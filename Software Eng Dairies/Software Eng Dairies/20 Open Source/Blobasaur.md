---
type: oss
status: to-try
created: 2026-11-22
repo: https://github.com/iamd3vil/blobasaur
language: Rust
license: AGPL-3.0
author: Sarat Chandra (iamd3vil), Zerodha
topics:
  - "[[Databases]]"
  - "[[Distributed Systems]]"
tags:
  - oss
  - zerodha
  - sqlite
  - redis-protocol
  - follow-up
---
# Blobasaur

## What it is
A sharded blob store that speaks the Redis protocol and keeps each shard in its own SQLite file. Zerodha uses it to store Kite's UI state (chart layouts, preferences) in place of SSDB. Full write-up: [[Distributed, high-performance blob storage on SQLite]].

## Why it's interesting
- Shows how far plain SQLite goes when you design around its limits: WAL allows one writer per file, so writes are spread across many shard files.
- Two-level routing: Redis Cluster hash slots pick the node, multi-probe consistent hashing picks the shard on that node.
- Group commit: one writer task per shard drains a queue and commits a batch in a single transaction.
- Optional async writes with an "inflight cache" so a `GET` right after a `SET` or `DEL` still sees the right answer.
- Being Redis-compatible meant existing Go and Python clients worked unchanged. The test suite runs a real go-redis client against a live server, over RESP2 and RESP3.
- It's small enough to read end to end (about 10 source files).

## What to read in the code
Files listed in the README's project structure:
1. `src/redis/protocol.rs`: the RESP parser, written with `nom`. Start here.
2. `src/server.rs`: accepting connections and dispatching commands.
3. `src/app_state.rs`: how a key is routed to a shard.
4. `src/shard_manager.rs`: write batching (group commit). This is the core idea.
5. `src/cluster.rs`: hash slots, `MOVED` replies and gossip.
6. `src/migration.rs`: offline resharding with `--verify`.

## Alternatives / compared to
- SSDB (what it replaced), SeaweedFS, MinIO/S3, ScyllaDB, RocksDB. The article explains why each lost.
- Similar idea elsewhere: Redis-protocol front ends on other engines, such as Kvrocks (RocksDB) and DragonflyDB.

## Follow-up
- [ ] Run it locally with Docker and hit it with `redis-cli` and `redis-benchmark`.
- [ ] Read `shard_manager.rs` and write down how a batch is formed and committed.
- [ ] Build my own smaller version (step 3 of [[Systems engineering learning path from Sarat's open source]]), then compare.
- [ ] Watch the repo for releases and new zerodha.tech posts about it.

## Notes from trying it
Link an experiment note here once I've played with it: [[]]
