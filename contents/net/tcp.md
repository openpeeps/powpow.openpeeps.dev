---
title: TCP
description: "Non-blocking TCP servers and clients on the powpow loop: connection pooling, corking, sendv, zero-copy file send and graceful shutdown."
keywords: ["powpow", "tcp", "networking", "server", "client"]
---

# TCP

`net/tcp.nim` provides non-blocking TCP servers and clients built on the event
loop: connection pooling, write buffering with corking, scatter-gather writes
(`sendv`), zero-copy file send, and graceful close-after-drain.

Runnable examples: [`examples/tcp_chat.nim`](https://github.com/openpeeps/powpow/blob/main/examples/tcp_chat.nim),
[`examples/tcp_client.nim`](https://github.com/openpeeps/powpow/blob/main/examples/tcp_client.nim),
[`examples/tcp_proxy.nim`](https://github.com/openpeeps/powpow/blob/main/examples/tcp_proxy.nim).

## TCP server

```nim
let loop = newLoop()

let server = newTcpServer(
  loop,
  onData  = proc(conn: Connection, data: openArray[byte]) =
    discard conn.send(data),          # echo
  onAccept = proc(conn: Connection) =
    echo "client connected: ", conn.getClientIp(),
  onClose  = proc(conn: Connection) =
    echo "client disconnected")

server.listen("0.0.0.0", 9010)        # or server.listenUnix("/tmp/sock")
loop.run()
```

`newTcpServer` returns a `TcpServer` with public fields `fd`, `loop`, `onClose`,
`connPool` and `maxConnections`.

## With its own loop

Omit the `loop` and the server creates and owns a private one:

```nim
let server = newTcpServer(
  onData  = proc(conn: Connection, data: openArray[byte]) =
    discard conn.send(data),          # echo
  onAccept = proc(conn: Connection) =
    echo "client connected: ", conn.getClientIp())

server.listen("0.0.0.0", 9010)
server.run()                          # blocks; frees the owned loop on exit
```

`server.getLoop()` exposes the loop when you need to share it (e.g. dial out
with `server.getLoop().connect(...)` or add timers). From a callback, call
`server.stop()` (or `server.close()`); `close()` frees the owned loop once
`run()` returns, immediately when called from outside `run()`. A borrowed loop
is never closed by the server. `server.start("0.0.0.0", 9010)` is shorthand
for `listen` + `run` (plus `startUnix` for UDS, POSIX).

`listenUnix(path, mode = 0o660)` binds a Unix domain socket instead of TCP
(POSIX). `injectFd(server, clientFd)` lets you hand a pre-accepted fd to the
server (used by the HTTP server's upgrade path).

## TCP client (async connect)

```nim
connect(loop, "example.com", 443,
  onConnect = proc(conn: Connection) = conn.send("GET / HTTP/1.1\r\n\r\n"),
  onData    = proc(conn: Connection, data: openArray[byte]) =
    stdout.write(cast[string](data)),
  onClose   = proc(conn: Connection) = echo "closed",
  onError   = proc(err: string) = echo "failed: ", err)
```

Hostnames are resolved **on the loop** via the [async DNS resolver](../core/dns);
DNS failure arrives through `onError`. `connectUnix(path, onConnect, onData,
onClose)` connects to a Unix socket (POSIX).

## Blocking connect with its own loop

Omit the `loop` for a self-contained client: it creates a private loop, drives
it until the connection closes (or the connect fails), then frees it. This is
the simple shape for scripts; concurrent clients sharing one loop should use
the `loop`-taking overload above instead.

```nim
connect("example.com", 443,
  onConnect = proc(conn: Connection) = conn.send("GET / HTTP/1.1\r\n\r\n"),
  onData    = proc(conn: Connection, data: openArray[byte]) =
    stdout.write(cast[string](data)),
  onClose   = proc(conn: Connection) = echo "closed",
  onError   = proc(err: string) = echo "failed: ", err)
# returns after the connection closes; the loop is already freed
```

`connectHe` (Happy Eyeballs) and `connectUnix` (POSIX) have the same blocking
overloads. A connection from any of these carries `ownsLoop = true`:
`conn.getLoop()` exposes the loop, `conn.run()` drives it, and `conn.close()`
stops/frees the owned loop.

## Sending data

```nim
conn.send(data: openArray[byte])      # int bytes sent
conn.send(data: string)               # int bytes sent
conn.sendv([(bufPtr, len), (bufPtr, len)])   # scatter-gather, one syscall
```

Writes are buffered and corked (`TCP_CORK`/`TCP_NOPUSH`) automatically, flushed
by the loop when the connection becomes writable.

## Zero-copy file send

`Connection` carries a sendfile state machine; the HTTP server uses this for
`sendFile`/`streamFile`:

```nim
conn.sendFileFd = fileFd              # open with openFileRead()
conn.sendFileOff = 0
conn.sendFileRemain = fileSize
while conn.continueSendFile():         # returns false when done
  discard
```

Under the **io_uring** backend the transfer is driven by `IORING_OP_SPLICE`
(file → pipe → socket) with no user-space copy; see the
[io_uring guide](../io_uring).

## Closing

```nim
conn.close()             # immediate close (SO_LINGER{0} fast shutdown)
conn.shutdown()          # stop the write side
conn.closeAfterDrain()   # close once the send buffer is fully flushed
conn.closeAndRelease()   # close and return buffers to the pool
```

Graceful shutdown: `closeAfterDrain` guarantees buffered data is sent before the
connection tears down. `close()` on an owned-loop connection additionally
stops/frees that loop (see above).

## Connection details

`Connection` public fields: `fd`, `loop`, `server`, `state`
(`ConnState`: `Connecting`, `Connected`, `Closing`, `Closed`), `clientIp`,
`data` (free per-connection pointer), `ssl`/`tlsState` (TLS layer).

```nim
conn.getClientIp()          # string
conn.getClientSockAddr()    # Sockaddr_storage
```

## Buffer pooling

Read buffers come from the loop pool:

```nim
let buf = loop.acquireBuf()
let n = sockRecv(conn.fd, buf, DefaultBufSize)
loop.releaseBuf(buf)
```

`MaxBufPoolSize` = 1024, `MaxConnPoolSize` = 1024.

## API reference

Full signatures: [TCP API](https://openpeeps.github.io/powpow/theindex.html). Related: [sockets](sockets),
[UDP](udp), [TLS](tls).
