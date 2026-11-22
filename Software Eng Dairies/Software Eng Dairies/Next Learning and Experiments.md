---
type: index
created: 2026-11-22
topics:
  - "[[Distributed Systems]]"
  - "[[Programming Languages]]"
  - "[[Databases]]"
  - "[[DevOps and Infra]]"
tags:
  - index
  - plan
  - career
---
# Next Learning and Experiments

What to learn next and what to build next, in priority order. The goal is to grow from backend/platform work toward systems and infra engineering, the kind of work in [[Systems engineering learning path from Sarat's open source]]. That note is the full build path (8 projects) and the concept list. This page is the short list for *me*: which of those to do first given where I already am, plus a few bridge projects that reuse what I already know.

Review this page once a month. Tick things off, move items between sections, and add a dated line to the review log at the bottom.

## Where I stand (as of 2026-11-22)

**Already strong. Build on these, don't re-learn them.**
- Distributed-systems *patterns* in production: Kafka with a transactional outbox, idempotency keys, lease-based workers, jittered bounded retries, at-least-once delivery.
- A scheduler of my own (MacBatch): shards, 600s leases reclaimed on failure, pull-based workers behind NAT, a throughput benchmark.
- Operating software on Kubernetes: Helm, Terraform, Istio, Prometheus, Grafana, Alertmanager, CI on GitHub Actions. Open-source maintainer (Kubeflow).
- Security basics in services: secrets in a vault, JWKS/JWT verification.
- Python, Java, TypeScript, AI agents and RAG, React.

**Gaps against the goal.**

| Gap                              | Why it matters for systems work                                                          | Evidence today                                                             |
| -------------------------------- | ---------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| No systems language (Go or Rust) | Almost everything in Sarat's path is Go or Rust; the Kubernetes and CNCF ecosystem is Go | Only Python, Java, TS, some C++                                            |
| Low-level networking             | Proxies, protocol servers and gateways all start at sockets and framing                  | Work is at the HTTP/REST/WebSocket layer                                   |
| Storage engine internals         | Blobasaur-style work needs WAL, fsync, B-tree vs LSM, compaction                         | Uses Postgres, Oracle, Elasticsearch as a client                           |
| Performance engineering          | Systems work is judged on p99 and allocations, not just throughput                       | One throughput benchmark (MacBatch); no profiling or latency percentiles   |
| Replication and consensus        | The step after "it works on one node"                                                    | Uses Kafka, hasn't built replication                                       |
| OS and Linux internals           | epoll, file descriptors, page cache, syscalls explain most performance surprises         | Not covered by a Mechanical Engineering degree; worth filling deliberately |
| Shipping binaries                | Static builds, cross-compilation, checksums                                              | Shipped an npm CLI and Docker images, not native binaries                  |

**Not a priority right now:** more AI/RAG, frontend, or new web frameworks. Keep them going through day-to-day work and Kubeflow, but they aren't where the growth is.

## Now (next 6 weeks)

**Decision: Go first, Rust later.** Go is faster to get productive in, it's the language of Kubernetes and Kubeflow (so it helps at work and in open source straight away), and it's what Sarat used for the proxy, DNS and decoder projects. Switch to Rust at the pub/sub step, the way he did. This answers the "Go or Rust" question in the learning path.

### Learn
- [ ] **Go, properly.** *A Tour of Go*, then *Learn Go with Tests*, then *Concurrency in Go* (Cox-Buday) for goroutines, channels, `context` and cancellation. *Done when* I can write a bounded worker pool with cancellation and backpressure without looking it up. [[Programming Languages]]
- [ ] **TCP and sockets.** Beej's Guide to Network Programming, then the TCP chapters of *TCP/IP Illustrated Vol. 1* (or *Computer Networking: A Top-Down Approach*). Cover the connection lifecycle, backlog, Nagle, keepalive, half-close and timeouts. Practise with `ss`, `tcpdump` and `strace` on a service I already run. [[Distributed Systems]]
- [ ] **TLS in practice.** Handshake, SNI, certificate chains, ACME (Let's Encrypt), mTLS. [[Security]]

### Build
- [ ] **Experiment 1: TCP/TLS proxy in Go** (learning path step 1). Port forward, then TLS termination with a real certificate, connection limits, idle timeouts, Prometheus `/metrics`. *Done when* it proxies Postgres or Redis traffic and survives a client that connects and never sends data. Then read Sarat's `proxy` and `mole` and note what he did differently. [[Distributed Systems]] [[Security]]
- [ ] **Bridge: a small Kubernetes controller in Go** with kubebuilder, e.g. one that watches a CRD and creates a Deployment. It turns existing Kubernetes knowledge into Go practice and is directly useful for Kubeflow (its operators are Go). [[DevOps and Infra]]

## Next (2 to 4 months)

### Learn
- [ ] **Storage internals.** *Designing Data-Intensive Applications* chapters 3, 5, 7 and 9; *Database Internals* (Petrov) part 1; the SQLite docs on WAL mode, `synchronous` and checkpoints. Learn what `fsync` really costs by measuring it. [[Databases]]
- [ ] **Profiling and honest benchmarking.** Go `pprof` and execution traces, flamegraphs, p50/p99/p999, coordinated omission (read Gil Tene's talk *How NOT to Measure Latency*, use `wrk2`). [[Developer Tools]]
- [ ] **OS fundamentals for servers.** *Operating Systems: Three Easy Pieces* (free) for processes, virtual memory, the page cache and I/O. Then epoll and how the Go netpoller uses it. [[Programming Languages]]

### Build
- [ ] **Experiment 2: mini Blobasaur, stages 1 to 4** (learning path step 3, the anchor project). RESP server `redis-cli` can talk to, then SQLite in WAL mode, then a writer goroutine per database with group commit, then sharding with consistent hashing. Benchmark with `redis-benchmark` after every stage and write the numbers down. [[Databases]] [[Distributed Systems]]
- [ ] **Experiment 3 (short, optional): DNS forwarder with cache** (learning path step 2). A weekend project for UDP and binary packet parsing. Skip it if Experiment 2 is going well. [[Distributed Systems]]
- [ ] **Bridge: profile MacBatch.** Add latency histograms (p50/p99 per task) next to the existing throughput number, profile the hot path, and write up one fix with before/after numbers. [[Distributed Systems]]

## Later (4 to 8 months)

### Learn
- [ ] **Replication and consensus.** The Raft paper (*In Search of an Understandable Consensus Algorithm*), then MIT 6.5840 (Distributed Systems). Its labs are in Go, which fits the plan. [[Distributed Systems]]
- [ ] **Rust.** *The Rust Book*, then *Rust for Rustaceans* or *Zero To Production*, then tokio's tutorial. [[Programming Languages]]
- [ ] **Kafka internals,** since I already run it: the log, segments, ISR replication, consumer group rebalancing. Read *Kafka: The Definitive Guide* chapters on internals. [[Distributed Systems]]

### Build
- [ ] **Experiment 4: MIT 6.5840 Raft labs** (leader election, log replication, persistence). Then, as a stretch, replicate one mini-Blobasaur shard with my own Raft. [[Distributed Systems]]
- [ ] **Experiment 5: pub/sub server on a NATS subset, in Rust** (learning path step 4). Wildcards, queue groups, and a deliberate slow-consumer policy. [[Distributed Systems]] [[Programming Languages]]
- [ ] **Experiment 6: ship them** (learning path step 8). Tagged releases with cross-compiled static binaries, checksums and Docker images through GitHub Actions. This reuses CI skills I already have. [[DevOps and Infra]]

After these: the SMTP relay, binary decoder and reverse proxy (learning path steps 5 to 7).

## Habits that run alongside
- **Write up every experiment** with numbers, as a short post on my site and a note in [[Tried It Log]]. First candidate: the Kafka outbox and lease design from work, written as a general pattern (no internal details).
- **Read one real codebase per project** before or after building it: Sarat's version, plus one well-known one (e.g. Redis `networking.c`, the Go `net` package, etcd's `raft`).
- **Contribute downstream in Go** once Experiment 1 is done: pick a "good first issue" in a Go infra project I already use (Kubeflow operators, Prometheus, NATS).

## How to use this page
1. When I start an item under **Build**, create a note in `30 Experiments` from the **Experiment** template, link it next to the checkbox here, and it will show up in [[Tried It Log]] and below.
2. When I finish a book or course under **Learn**, create a note in `10 Learnings` from the **Learning** template and link it here.
3. Tick the box only when its *done when* is true.

### Experiments in progress
```base
filters:
  and:
    - type == "experiment"
    - status == "in-progress"
    - file.folder != "90 Templates"
views:
  - type: table
    name: In progress
    order:
      - file.name
      - tried
      - topics
```

## Review log
- 2026-11-22: First version, built from my resume and [[Systems engineering learning path from Sarat's open source]]. Chose Go first.

## Related
- [[Systems engineering learning path from Sarat's open source]]
- [[Tried It Log]]
- [[Blobasaur]]
- [[4th Cross Labs]]
- [[Home]]
