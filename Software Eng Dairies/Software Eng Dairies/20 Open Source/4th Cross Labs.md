---
type: oss
status: to-try
created: 2026-11-22
repo: https://fourthcross.tech/
language: Go, Rust, others
license: various
topics:
  - "[[Distributed Systems]]"
  - "[[DevOps and Infra]]"
  - "[[Developer Tools]]"
tags:
  - oss
  - zerodha
  - directory
  - follow-up
---
# 4th Cross Labs

## What it is
Zerodha's directory of open source projects, named after the street in JP Nagar, Bengaluru, where the team started. It lists about 19 featured and 118 more projects, sorted by stars, many marked "Used in production at Zerodha". Sarat's (iamd3vil) [[Blobasaur]], Hedwig and shelf are on it.

## Why it's interesting
- A reading list of real production tools from one small, strong engineering team, mostly Go and Rust, mostly self-hosted.
- Shows the same habits across people: small focused tools, plain config files, single binaries, good READMEs.
- Good place to pick a "next repo to read" once I've finished one from [[Systems engineering learning path from Sarat's open source]].

## Worth a look first
- listmonk (knadh): self-hosted newsletter manager. A complete Go + Vue product, good for reading a whole app.
- dungbeetle (zerodha): distributed job server for heavy SQL read jobs.
- kaf-relay (zerodha): Kafka replication relay.
- logchef: log analytics UI for ClickHouse.
- koanf (knadh): Go config library, a nice small library to read.
- fastglue (zerodha): thin HTTP layer on fasthttp.
- doggo (mr-karan): DNS client CLI. Pairs well with Sarat's `danse`.
- hedwig (iamd3vil): SMTP server in Rust.

## Notes from trying it
Link an experiment note here once I've played with something from the list: [[]]
