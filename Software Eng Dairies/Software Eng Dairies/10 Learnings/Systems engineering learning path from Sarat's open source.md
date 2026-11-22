---
type: learning
status: reading
created: 2026-11-22
source: https://github.com/iamd3vil
author: Sarat Chandra (iamd3vil), Zerodha
topics:
  - "[[Distributed Systems]]"
  - "[[Databases]]"
  - "[[DevOps and Infra]]"
  - "[[Programming Languages]]"
tags:
  - learning
  - career
  - learning-path
  - zerodha
---
# Systems engineering learning path from Sarat's open source

## TL;DR
Sarat Chandra works on financial systems at Zerodha. Most of his open source is one move repeated: take a protocol that already exists (Redis, NATS, SMTP, DNS, TCP, FIX FAST), write a small, fast server or tool that speaks it, run it in production, and measure it. To grow toward that kind of work, I should do the same thing in order of difficulty: proxies, then a Redis-compatible store, then pub/sub, then mail, then binary decoders and a gateway. Each step should be shipped like he ships: a binary, a README, metrics and a benchmark.

Sources: his GitHub repos (READMEs of the projects below), [sarat.dev](https://sarat.dev/), [the Blobasaur article](https://zerodha.tech/blog/high-performance-distributed-blob-storage/), and Zerodha's project directory [[4th Cross Labs]]. Anything marked (inferred) is my reading, not something he wrote.

## Who he is
- Software engineer in Bengaluru, working on financial systems at Zerodha.
- Has written Go and Elixir professionally since 2015 and uses Rust for personal projects (per sarat.dev). His repos follow the same arc: Elixir 2016-2019, Go 2019-2023, mostly Rust since 2024.
- Says realtime and distributed systems are what keep him interested.
- Self-hosts a lot (Nextcloud, VPN, Caddy, Jellyfin). His blog is short practical posts, e.g. using Go WaitGroups without leaky abstractions.

## The pattern in his work (inferred)
1. **Speak an existing protocol.** Blobasaur speaks Redis, Wisp speaks NATS, Hedwig speaks SMTP, Danse speaks DNS. Existing clients work unchanged, so adoption is nearly free.
2. **Build because a real need showed up.** SSDB was unmaintained, so Blobasaur. Multicast market data couldn't cross networks, so Magicportal. FIX FAST feeds needed decoding, so gofastdecoder.
3. **Keep it small and boring to run.** One static binary, a TOML/YAML config, SQLite instead of a cluster when possible.
4. **Measure everything.** Prometheus `/metrics`, health checks, p50/p99 latency, benchmarks with test conditions stated (and caveats like "ran on tmpfs").
5. **Do the operational parts.** Graceful shutdown, config reload on SIGHUP, slow-consumer limits, vacuum and TTL cleanup, offline migration with `--verify`.
6. **Ship properly.** Release binaries with checksums, Docker images, docs sites. He even wrote his own release tool (rlsr) and version bumper (verbump).

## Project map: what each one teaches

| Project | What it is | What it teaches |
|---|---|---|
| [[Blobasaur]] (Rust) | Redis-protocol blob store on sharded SQLite | RESP parsing, storage engine limits (WAL single writer), group commit, consistent hashing, Redis Cluster slots, TTL, compression, read-your-writes with async writes, migrations |
| hedwig (Rust) | SMTP relay server | SMTP state machine, persistent on-disk queue, retries, TLS/STARTTLS, DKIM signing, MTA-STS, per-domain rate limits |
| wisp (Rust) | Pub/sub server on the NATS wire protocol | Subject matching (`*`, `>`), queue groups, backpressure and slow-consumer policy, write batching (`writev`), PING/PONG liveness, mTLS, graceful drain |
| raahi (Rust) | Reverse proxy and API gateway on Cloudflare's Pingora | L7 routing, load balancing, health checks, TLS by SNI, ACME certificates, hot config swap, rate limits, JWT auth, WASM plugins, path-traversal safety |
| proxy, mole (Go) | TCP/TLS proxy with Let's Encrypt; SSH tunnels from config | Sockets, `io.Copy` pumping, TLS termination, ACME DNS-01, tunnelling |
| danse (Go) | Local DNS resolver that forwards over DoH/DoT/DNSCrypt | DNS packet format, UDP servers, caching by TTL, upstream load balancing |
| magicportal (Rust) | Forwards multicast UDP over NATS | Multicast, UDP, bridging transports (exchange market data is multicast) |
| gofastdecoder (Go) | FIX FAST 1.1/1.2 decoder with code generator | Binary market-data protocols, stop-bit encoding, presence maps, code generation, zero-allocation hot paths |
| binserde, csvgen, gandalf (Go) | Code generators for binary serde, CSV parsing, struct validation | `go generate`, avoiding reflection for speed |
| rlsr, verbump (Rust) | Release automation and semver tagging | Cross-compiling, CI pipelines, checksums, shipping software |
| smtpblackhole, smtpbench | SMTP sink and SMTP benchmark tool | Building your own load generator to test your server |
| nats_ex, rifle, twirpex (Elixir) | NATS client, HTTP client with pooling, Twirp RPC | Writing clients: connection pooling, protocol framing, RPC |

## Concepts to study
Grouped by area, roughly in the order the projects below need them.

**Networking**
- TCP: connection lifecycle, backlog, Nagle, keepalive, half-close, timeouts and deadlines.
- UDP and multicast. DNS message format.
- TLS: handshake, SNI, certificates, ACME, mTLS.
- Framing a protocol on a byte stream: length-prefixed, line-based (RESP, NATS, SMTP), binary with stop bits (FAST).

**Concurrency and performance**
- One async runtime well: Go goroutines/channels, or Rust tokio. Bounded queues and backpressure.
- Batching: group commit, `writev`, pipelining.
- Allocation and copying: buffer reuse, zero-copy parsing. Profiling: Go `pprof`, Rust `perf` and flamegraphs.
- Latency percentiles (p50/p99/p999) and how to benchmark honestly.

**Storage**
- SQLite internals: WAL mode, `synchronous`, checkpoints, page freelist, vacuum.
- B-trees vs LSM trees. fsync and what durability actually costs.
- Partitioning: hash vs range, consistent hashing, rebalancing.

**Distributed systems**
- Replication, failover and consensus (Raft) at the concept level. Blobasaur hasn't built these yet, which is a good gap to study.
- Gossip and cluster membership. Idempotency and retries. Safe migrations (dual-write, backfill, read-fallback, cutover).

**Operating software**
- Metrics, health and readiness checks, structured logs.
- Graceful shutdown, config reload, resource limits.
- Release engineering: static builds, cross-compilation, Docker images.

## Build-it-yourself path
One project every 3-6 weeks alongside work. For each one, write a README with config and a benchmark section, expose `/metrics`, and log the build in [[Tried It Log]]. Use Go for steps 1-3 if it's more familiar, then move to Rust, the way he did. After each step, read his version and note what he did differently.

1. **TCP/TLS proxy.** Forward a port, then add TLS termination with a real certificate, connection limits and timeouts. *Done when* it proxies Postgres or Redis traffic and survives a client that never sends data. *Compare with:* `proxy`, `mole`.
2. **DNS forwarder with cache.** Listen on UDP 53, forward to a DoH upstream, cache by TTL. *Done when* `dig` works through it and repeat lookups are served from cache. *Compare with:* `danse`.
3. **Mini Blobasaur (the big one).** Do it in stages:
   1. RESP parser and an in-memory `GET`/`SET`/`DEL` server that `redis-cli` can talk to.
   2. Store in SQLite (WAL). Benchmark with `redis-benchmark`.
   3. Add a writer task per database with group commit. Measure the throughput change.
   4. Shard across N SQLite files with consistent hashing. Measure again.
   5. Add TTL with a background cleanup that doesn't stall writes, and zstd compression.
   6. Stretch: two nodes with hash slots and `MOVED` replies.
   *Done when* a go-redis or redis-py client passes a small test suite against it. *Compare with:* [[Blobasaur]] and [[Distributed, high-performance blob storage on SQLite]].
4. **Pub/sub server on a NATS subset.** `CONNECT`, `PUB`, `SUB`, `UNSUB`, `PING`/`PONG`, wildcards, queue groups. Then decide what happens to a slow subscriber. *Done when* the official NATS CLI can publish and subscribe through it, and a slow consumer can't take down the server. *Compare with:* `wisp`.
5. **SMTP sink, then relay.** First a server that accepts mail and drops it, plus a small load generator. Then a relay with an on-disk queue, retries and DKIM signing. *Compare with:* `smtpblackhole`, `smtpbench`, `hedwig`.
6. **Binary protocol decoder with code generation.** Pick a simple binary format (or FAST), generate a decoder from a schema, and get the hot path to zero allocations. *Compare with:* `binserde`, `gofastdecoder`.
7. **Reverse proxy with hot reload.** HTTP routing by host and path, round-robin with health checks, rate limiting, config stored in SQLite and swapped without restart. *Compare with:* `raahi`.
8. **Ship them.** Once two or three exist, set up tagged releases with cross-compiled binaries, checksums and Docker images. *Compare with:* `rlsr`.

## Reading list
- The Blobasaur article: [[Distributed, high-performance blob storage on SQLite]].
- Protocol specs as you reach each step: Redis RESP and Redis Cluster spec, NATS client protocol, RFC 5321 (SMTP), RFC 1035 (DNS).
- SQLite docs on WAL mode and `PRAGMA` settings.
- *Designing Data-Intensive Applications* (Kleppmann) for the storage and distributed-systems chapters.
- [zerodha.tech](https://zerodha.tech/) blog, and [[4th Cross Labs]] for more repos from the same team.

## How I'd use it
- Treat step 3 as the anchor project. It's close to what he built for real, and it touches storage, networking, concurrency and benchmarking at once.
- Write up each step as a short post with numbers. That's the habit that makes this kind of work visible.

## Questions / follow-ups
- ~~Pick Go or Rust for step 1.~~ Decided: Go first, Rust from step 4. See [[Next Learning and Experiments]].
- Read `src/shard_manager.rs` in Blobasaur before starting step 3.3.
- sarat.dev lists a "Blobnom" SQLite blob store; it's probably an earlier name of Blobasaur (inferred, not confirmed).

## Related
- [[Next Learning and Experiments]] (my prioritized version of this path)
- [[Blobasaur]]
- [[4th Cross Labs]]
- [[Distributed, high-performance blob storage on SQLite]]
- [[Tried It Log]]
