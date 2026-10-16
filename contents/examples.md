---
title: Examples
description: "Every runnable powpow example on GitHub: HTTP, WebSocket, TCP, UDP, DTLS, TLS, UDS, clients, timers, signals and rate limiting."
keywords: ["powpow", "examples", "demos", "tutorial"]
---

# Examples

Every example lives in
[`examples/`](https://github.com/openpeeps/powpow/tree/main/examples)
and is runnable as-is. Clone the library repo, then `nim c -r` any file
below. Ignore prebuilt binaries in `examples/`; only the `.nim` sources
matter.

> Each row links to its source on GitHub. The guides throughout these
> docs explain the interesting parts tour-style with one snippet per
> feature.

## HTTP servers

| Example | Demo | Source |
|---|---|---|
| `httpserver.nim` | Tiny HTTP/1.1 server, manual routing | [source](https://github.com/openpeeps/powpow/blob/main/examples/httpserver.nim) |
| `httpserver_threads.nim` | One loop per core via `SO_REUSEPORT` | [source](https://github.com/openpeeps/powpow/blob/main/examples/httpserver_threads.nim) |
| `httpserver_multiport.nim` | One server on several ports | [source](https://github.com/openpeeps/powpow/blob/main/examples/httpserver_multiport.nim) |
| `httpserver_ktls.nim` | HTTPS with kernel-TLS offload probe | [source](https://github.com/openpeeps/powpow/blob/main/examples/httpserver_ktls.nim) |
| `http2server.nim` | HTTP/2 over TLS (`h2`) + cleartext (`h2c`) | [source](https://github.com/openpeeps/powpow/blob/main/examples/http2server.nim) |
| `http2server_threads.nim` | Multi-threaded HTTP/2 via `SO_REUSEPORT` | [source](https://github.com/openpeeps/powpow/blob/main/examples/http2server_threads.nim) |
| `static_server.nim` | Static site + CORS + JSON API | [source](https://github.com/openpeeps/powpow/blob/main/examples/static_server.nim) |
| `stream_server.nim` | `streamFile` / `sendFile` / `serveFile`, Range and resume | [source](https://github.com/openpeeps/powpow/blob/main/examples/stream_server.nim) |
| `stream_server_ktls.nim` | Streaming and downloads over TLS with kTLS | [source](https://github.com/openpeeps/powpow/blob/main/examples/stream_server_ktls.nim) |
| `upload_server.nim` | Raw-body and multipart uploads | [source](https://github.com/openpeeps/powpow/blob/main/examples/upload_server.nim) |
| `uds_server.nim` | HTTP over a Unix domain socket | [source](https://github.com/openpeeps/powpow/blob/main/examples/uds_server.nim) |
| `tls_server.nim` | HTTPS with an embedded self-signed cert | [source](https://github.com/openpeeps/powpow/blob/main/examples/tls_server.nim) |

## Clients

| Example | Demo | Source |
|---|---|---|
| `httpclient.nim` | Sync + async HTTP client, pooling, UDS | [source](https://github.com/openpeeps/powpow/blob/main/examples/httpclient.nim) |

## WebSocket

| Example | Demo | Source |
|---|---|---|
| `wsserver.nim` | Standalone WebSocket server | [source](https://github.com/openpeeps/powpow/blob/main/examples/wsserver.nim) |
| `wsclient.nim` | High-level client: reconnect, sendMessage | [source](https://github.com/openpeeps/powpow/blob/main/examples/wsclient.nim) |
| `wsupgrade.nim` | HTTP and WebSocket on one port | [source](https://github.com/openpeeps/powpow/blob/main/examples/wsupgrade.nim) |
| `ws_chat.nim` | Multi-client chat with broadcast | [source](https://github.com/openpeeps/powpow/blob/main/examples/ws_chat.nim) |

## TCP and proxy

| Example | Demo | Source |
|---|---|---|
| `tcp_chat.nim` | Multi-client chat room on raw TCP | [source](https://github.com/openpeeps/powpow/blob/main/examples/tcp_chat.nim) |
| `tcp_client.nim` | Interactive stdin client for the chat | [source](https://github.com/openpeeps/powpow/blob/main/examples/tcp_client.nim) |
| `tcp_proxy.nim` | Low-level reverse proxy with connect buffering | [source](https://github.com/openpeeps/powpow/blob/main/examples/tcp_proxy.nim) |
| `proxyserver.nim` | High-level `ProxyServer` reverse proxy | [source](https://github.com/openpeeps/powpow/blob/main/examples/proxyserver.nim) |

## UDP and DTLS

| Example | Demo | Source |
|---|---|---|
| `udp_echo.nim` | UDP echo server plus ping client mode | [source](https://github.com/openpeeps/powpow/blob/main/examples/udp_echo.nim) |
| `dtls_echo.nim` | DTLS 1.2 echo with cookie exchange and MTU splitting | [source](https://github.com/openpeeps/powpow/blob/main/examples/dtls_echo.nim) |

## Core features

| Example | Demo | Source |
|---|---|---|
| `timers_scheduler.nim` | Timer wheel: one-shot, interval, deferred, idle | [source](https://github.com/openpeeps/powpow/blob/main/examples/timers_scheduler.nim) |
| `fswatch.nim` | File system watcher events | [source](https://github.com/openpeeps/powpow/blob/main/examples/fswatch.nim) |
| `os_signals.nim` | Graceful shutdown on SIGINT / SIGTERM | [source](https://github.com/openpeeps/powpow/blob/main/examples/os_signals.nim) |
| `signal_bus.nim` | In-process pub/sub `SignalRelay` | [source](https://github.com/openpeeps/powpow/blob/main/examples/signal_bus.nim) |
| `stream_pipe.nim` | `IoStream` socketpair echo with EOF | [source](https://github.com/openpeeps/powpow/blob/main/examples/stream_pipe.nim) |
| `ratelimit_server.nim` | Sliding-window per-IP rate limiting | [source](https://github.com/openpeeps/powpow/blob/main/examples/ratelimit_server.nim) |

`devcert.nim` is a shared helper (throwaway self-signed cert/key), not a
standalone demo.

## Port map

| Port | Example |
|---|---|
| 9000 | `httpserver`, `httpserver_threads`, `httpserver_multiport`, `wsupgrade`, `upload_server` |
| 9001 | `wsserver`, `httpserver_multiport` (second port), `proxyserver` backend |
| 9002 | `stream_server` (needs the ~2.76 GB `.webm`) |
| 9003 | `ratelimit_server` |
| 9004 | `static_server` |
| 9005 | `signal_bus` |
| 9006 | `ws_chat` |
| 9007 | `os_signals` |
| 9010 | `tcp_chat` and `tcp_client` |
| 9011 | `udp_echo` |
| 9012 | `dtls_echo` |
| 9020 / 9021 | `tcp_proxy`, `proxyserver` and their backends |
| 9040 | `http2server` (`h2` over TLS by default, `--h2c` for cleartext) |
| 9443 | `tls_server` |
| 9444 | `httpserver_ktls` |
| none | `timers_scheduler`, `fswatch`, `stream_pipe`, `httpclient` |
| UDS path | `uds_server` (`/tmp/powpow.sock`) |

## Quick smoke test

```bash
# HTTP server
curl http://localhost:9000/
curl http://localhost:9000/api/echo -d 'Hello powpow!'

# WebSocket
websocat ws://localhost:9001          # or: npx wscat -c ws://localhost:9001

# TCP chat
nc 127.0.0.1 9010

# UDP
echo "hello udp" | nc -u -w1 127.0.0.1 9011
nim c -r examples/udp_echo.nim -- --client

# TLS and DTLS
curl -k https://localhost:9443/hello
nim c -r examples/dtls_echo.nim -- --client

# HTTP/2 (cleartext)
curl --http2-prior-knowledge http://localhost:9040/hello

# UDS
curl --unix-socket /tmp/powpow.sock http://localhost/hello

# Rate limiting (429 after the 5th request)
for i in $(seq 6); do curl -s -o /dev/null -w "%{http_code}\n" http://localhost:9003/; done

# Uploads
curl -X POST http://localhost:9000/upload/raw --data-binary @bigfile.bin
curl -X POST http://localhost:9000/upload/stream -F "file=@bigfile.bin"
```
