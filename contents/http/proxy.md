---
title: Proxy
description: "TCP reverse proxying with powpow: the high-level ProxyServer and the low-level hand-rolled pattern."
keywords: ["powpow", "proxy", "reverse proxy", "tcp", "load balancer"]
---

# Proxy

Two ways to proxy TCP with powpow:

1. **`proto/proxyserver.nim` (`ProxyServer`)** — high level. Owns its
   `Loop` and frontend `TcpServer`s; you set the upstream and wire
   notification callbacks. Forwarding is automatic.
2. **Hand-rolled (`newTcpServer` + `connect`)** — the
   [`examples/tcp_proxy.nim`](https://github.com/openpeeps/powpow/blob/main/examples/tcp_proxy.nim)
   pattern: explicit pair table, pending buffering, manual teardown. Use it
   when you need per-byte control.

Runnable examples:
[`examples/proxyserver.nim`](https://github.com/openpeeps/powpow/blob/main/examples/proxyserver.nim)
(high level),
[`examples/tcp_proxy.nim`](https://github.com/openpeeps/powpow/blob/main/examples/tcp_proxy.nim)
(low level).

## High-level proxy

```nim
import powpow/proto/proxyserver

let proxy = newProxyServer("127.0.0.1", 9001)   # static upstream
# or: let proxy = newProxyServer()              # then setUpstream later
# or: let proxy = newProxyServer(loop)          # share an existing Loop

proxy
  .onConnect(proc(pair: ProxyPair) {.gcsafe.} =
    echo "client accepted, id=", pair.id)
  .onUpstreamConnect(proc(pair: ProxyPair) {.gcsafe.} =
    echo "proxied to upstream")
  .onData(proc(pair: ProxyPair, dir: ProxyDir,
               data: openArray[byte]) {.gcsafe.} =
    echo dir, " ", data.len, " bytes")
  .onClose(proc(pair: ProxyPair, dir: ProxyDir) {.gcsafe.} =
    echo "closed from ", dir)
  .onError(proc(pair: ProxyPair, err: string) {.gcsafe.} =
    echo "error: ", err)

proxy.setUpstream("127.0.0.1", 9001)
proxy.listen("0.0.0.0", 9020)           # additive, multi-port ready
proxy.start(Port(9020))                 # blocks: listen + loop.run()
```

Behavior: bytes flow both ways; client bytes arriving before the upstream
connects are buffered and flushed on connect; closing either side tears
down the pair. Callbacks are notifications — forwarding already happened.
Inject with `pair.client.send` / `pair.upstream.send` inside a callback.

Direction sugar: `onClientData` / `onUpstreamData` filter `onData` for you.
`ProxyDir` is `FromClient` | `FromUpstream`. `teardownByUpstreamFd` covers
the rare case where you only kept the upstream fd.

## Caps

```nim
proxy.setMaxPending(64 * 1024)      # per-pair buffered bytes while connecting
proxy.setMaxConnections(4096)       # frontend connection cap
```

Without an upstream configured, clients get an `onError` ("no upstream
configured") instead of hanging.

## Low-level pattern

`tcp_proxy.nim` accepts on `:9020`, dials the backend on `:9021`, pipes
bytes both ways, and buffers anything arriving before the upstream is
ready. Test it with `nc 127.0.0.1 9020`. Reach for this shape when the
proxy must inspect, rewrite, or route bytes rather than just forward them.

## Run it

```bash
# backend on :9001, proxy on :9020
nim c -r examples/proxyserver.nim
nc 127.0.0.1 9020        # type a line; the backend echoes it back
```

## API reference

Full signatures:
[generated API reference](https://openpeeps.github.io/powpow/theindex.html).
Related: [TCP](../net/tcp.md), [HTTP server](server.md),
[concurrency](../concurrency.md).
