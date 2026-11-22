---
title: Distributed, high-performance blob storage on SQLite
source: https://zerodha.tech/blog/high-performance-distributed-blob-storage/
author: Sarat Chandra (Zerodha)
type: learning
format: article
published: 2026-11-19
created: 2026-11-22
reviewed: 2026-11-22
topics:
  - "[[Databases]]"
  - "[[Distributed Systems]]"
tags: [learning, engineering, storage, sqlite, rust, distributed-systems, zerodha, blob-storage]
status: done
---

## TL;DR
Zerodha replaced SSDB with **Blobasaur**, an open source (AGPLv3) Rust server that speaks the Redis protocol and stores blobs in sharded SQLite files (WAL mode). Redis Cluster slots route requests to nodes, and consistent hashing routes them to shards within a node. Production p99 is about 2 ms on three small nodes.

## Context & problem
- Kite (~20M customers) saves UI state (chart layouts, indicator templates, preferences) as opaque JSON blobs, from a few KB to several MB.
- History: CouchDB, then SSDB (C++, Redis protocol) for years.
- SSDB problems: unmaintained, C++ is outside the team's stack so forking wasn't viable, single-node only, and latency degraded on large blobs (capped at 1 MB at one point).
- Requirements: simple KV API, multi-MB blobs at thousands to tens of thousands of req/s (bursty with market volatility), horizontal scaling, compression, Redis protocol compatibility so existing Go/Python clients keep working, single-digit ms latency, TTL support. Relaxed durability is acceptable because losing a recent layout tweak is low-stakes.

## Architecture
- **Routing level 1 (machine):** `CRC16(key) % 16384` -> hash slot -> node. Nodes own configured slot ranges, discover each other via gossip, and reply with `MOVED` for wrong-node requests (Redis Cluster protocol).
- **Routing level 2 (shard):** multi-probe consistent hashing picks one of N SQLite files on that node.
- **Why shards:** WAL mode allows one writer per file, and writes are about 50% of production traffic.
- **Per shard:** a writer task fed by a bounded queue does *group commit* (drain a batch, one `BEGIN IMMEDIATE ... COMMIT`). A pool of read-only connections serves reads concurrently.
- **Schema:** `blobs(key PK, data BLOB, created_at, updated_at, expires_at, version)`. `HSET` goes to a per-namespace table `blobs_<namespace>`. A partial index on `expires_at` (only rows with a TTL) keeps the index small.
- **TTL:** expired rows are filtered at read time and physically deleted by a background worker.
- **Stack:** Rust, tokio, sqlx, rusqlite (SQLite C API directly). The authors cite problems with Go's CGo SQLite driver: static builds, OS thread pinning, GC pressure on multi-MB buffers.

## Key decisions & trade-offs
- **SQLite over a purpose-built blob store** -> embedded, battle-tested, one file per shard, easy to inspect with the `sqlite3` CLI. Gave up: live resharding and replication, which they had to build around it.
- **Multiple shards per node** -> removes the single-writer bottleneck. Gave up: more files to manage and an offline reshard step.
- **Multi-probe consistent hashing** -> going from 4 to 5 shards moves ~20% of keys instead of ~80% with `hash % n`, with no virtual nodes needed.
- **`synchronous = NORMAL` plus optional `async_write`** -> faster writes. Gave up: a crash can lose recently acknowledged writes (the database stays consistent). Acceptable for UI preferences, not for anything critical.
- **Compression (zstd in production)** -> in one hour of production HSET traffic, 63.6 KiB average payload stored as 5.5 KiB (~11x); p99 935 KiB stored as 59.3 KiB (~16x). Fewer pages written means less I/O.
- **Redis protocol compatibility** -> migration only needed a switch from a single-node Redis client to a Redis Cluster client.
- **Rejected alternatives:**
  - ScyllaDB: lower throughput and higher resource use on large blobs; tombstones from overwrites and TTLs need tuning.
  - PostgreSQL: BYTEA was poor for performance and disk efficiency.
  - YugabyteDB: not designed for blobs; too feature-heavy.
  - RocksDB: not built for high-throughput multi-MB writes.
  - Ceph: operational complexity (not tried).
  - SeaweedFS: closest. Disk usage grows under overwrite-heavy use because it appends before compaction, blobs have their own IDs and need a separate key mapping, there is no per-blob TTL, and setup is complex.
  - S3/MinIO: high latency for small blobs, per-request cost (writes are 50% of traffic), coarse TTLs.
  - Raw files: would mean rebuilding atomic writes, batching and TTL indexing.

## Numbers
| Test (64 conns, 64 KiB JSON, 100k keys, tmpfs, batch_size=50) | 1 shard | 16 shards |
|---|---|---|
| HSET only | 7,740 ops/s, p99 10.7 ms | 25,324 ops/s, p99 8.3 ms |
| 50% HSET / 50% HGET | 14,227 ops/s, p99 10.5 ms | 29,538 ops/s, p99 8.5 ms |
| HGET only | 67,128 ops/s, p99 2.1 ms | 70,884 ops/s, p99 2.3 ms |

- Sharding raised write throughput 3.3x and cut median write latency from 8.2 ms to 2.1 ms. Reads barely changed because WAL already allows concurrent readers.
- **Caveat:** benchmark ran on tmpfs on a shared 16-core laptop, not production disks.
- **Production:** 3 nodes, each 4 vCPU / 8 GB. During the trading session (09:00-15:30): p99 about 2 ms on every node, p50 250-550 us. Per command p99: HSET ~2 ms, HGET ~1.4 ms, HDEL mostly under 1 ms.

## Failure modes, limits & lessons
- **Deletes don't shrink files:** SQLite keeps freed pages on a freelist. Fix: `PRAGMA auto_vacuum = INCREMENTAL` on each shard, a `blobasaur shard vacuum` command, then `wal_checkpoint(TRUNCATE)`.
- **TTL cleanup caused latency spikes:** p99 writes normally stayed under 5 ms but hit 30-50 ms. A background connection ran one large DELETE and contended for the write lock. Fix: run cleanup through the shard's writer and chunk deletes (`cleanup_chunk_size`, default 1,000 rows).
- **Read-your-writes with `async_write`:** a pending in-memory map is checked first. An early bug let `DEL` then `GET` return the stale value. Fix: explicit tombstones in the pending map.
- **Not implemented yet:** automatic failover and live slot rebalancing. Slot assignment is static config. Resharding is offline (`blobasaur shard migrate 4 5 --verify`, then edit `num_shards`).
- SQLite beats raw files for small blobs (up to ~35% faster by their claim), but raw files win above ~100 KiB. Compression keeps 99% of values well below that.
- **Migration path that worked:** new workload first, then dual-write plus backfill with read-fallback to SSDB, then cutover. A month of dual-writes showed "zero discrepancies" before SSDB was turned off.

## Takeaways
- For a write-heavy SQLite service, shard into multiple files. WAL's single writer is the ceiling, not SQLite's read speed.
- Group commit (batch many writes into one transaction) is the main throughput lever for small writes.
- Compress repetitive JSON before storing. It cuts I/O as well as disk.
- Choosing a wire-compatible protocol (Redis) makes migration nearly free for clients. Dual-write plus read-fallback is a safe cutover pattern.
- Plan for the unglamorous parts: vacuuming, TTL cleanup contention and checkpointing cost them more than the core design.

## Open questions
- (inferred) With no replication or failover described, a node loss likely makes its slots unavailable. How do they handle that today?
- No cluster-wide throughput or multi-node scaling numbers are given.
- How does it behave on real disks under the same load as the tmpfs benchmark?
- Worth reading the repo: https://github.com/iamd3vil/blobasaur (group commit and the pending-write map in particular).

## Related
- [[Welcome]]
