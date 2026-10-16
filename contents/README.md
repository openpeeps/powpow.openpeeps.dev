---
title: powpow documentation
description: "Index of the powpow documentation: guides, examples, API reference, performance, security and testing."
keywords: ["powpow", "documentation", "index"]
---

# powpow documentation

Welcome to the powpow documentation. powpow is a high-performance, event
notification library for Nim: a low-level event loop and timer wheel, plus
higher-level abstractions for building TCP/UDP, HTTP/1.1, WebSocket, TLS and
multi-threaded servers.

## Getting started

@overview.md

@getting-started.md

## Guides

### Core

@core/event-loop.md

@core/signals.md

@core/fswatch.md

@core/streams.md

@core/dns.md

### Networking

@net/sockets.md

@net/tcp.md

@net/udp.md

@net/tls.md

@net/ktls.md

@net/dtls.md

### HTTP

@http/parser.md

@http/requests.md

@http/server.md

@http/client.md

@http/http2.md

@http/proxy.md

@http/static-files.md

@http/multipart.md

@http/rate-limiting.md

### Protocols & concurrency

@websocket.md

@concurrency.md

## Reference

@examples.md

Full per-module signatures live in the
[generated API reference](https://openpeeps.github.io/powpow/theindex.html).

@performance.md

@security.md

@testing.md

## Also in the library repository

- [`plans/`](https://github.com/openpeeps/powpow/tree/main/plans) — development plans and roadmap (not user docs)
- [`audit/`](https://github.com/openpeeps/powpow/tree/main/audit) — the security audit report and its reproducers
