---
title: UDP
description: "Bound and connected UDP sockets on the powpow loop: bindUdp, connectUdp, sendTo and send."
keywords: ["powpow", "udp", "datagram", "socket"]
---

# UDP

`net/udp.nim` provides non-blocking UDP sockets in two modes: **bound**
(server-style, receives from anyone) and **connected** (client-style, scoped to
one peer).

Runnable example: [`examples/udp_echo.nim`](https://github.com/openpeeps/powpow/blob/main/examples/udp_echo.nim).

## Bound UDP socket (server)

```nim
let sock = bindUdp(loop, "0.0.0.0", 9011,
  proc(sender: Sockaddr_storage; data: openArray[byte]) =
    sock.sendTo(data, sender)     # echo back to the sender
)
```

The callback receives the sender's address and the datagram bytes. `OnUdpData =
proc(sender: Sockaddr_storage; data: openArray[byte]) {.closure.}`

Omit the `loop` and the socket creates and owns a private one:

```nim
let sock = bindUdp("0.0.0.0", 9011,
  proc(sender: Sockaddr_storage; data: openArray[byte]) =
    sock.sendTo(data, sender))

sock.run()      # blocks, driving the owned loop
sock.close()    # frees the socket and the owned loop
```

`sock.getLoop()` exposes the loop when you need to share it (e.g. a second
socket or timers on the same loop). From a callback, `sock.stop()` (or
`sock.close()`) returns from `run()` and frees the owned loop. A borrowed loop
is never closed by the socket.

## Connected UDP socket (client)

```nim
let sock = connectUdp(loop, "127.0.0.1", 9011, onData)
sock.send("ping")
```

A connected UDP socket can `send` (no address) and only receives datagrams from
its peer. `connectUdp("127.0.0.1", 9011, onData)` without a loop creates and
owns a private loop, with the same `run`/`stop`/`close`/`getLoop` lifecycle as
a bound socket above.

## Sending

```nim
sock.sendTo(data, "example.com", 1234)    # resolve each call (address form)
sock.sendTo(data, addrBuf)                # use a cached Sockaddr_storage
sock.send(data)                           # connected sockets only
```

The address forms re-resolve per datagram (see [async DNS](../core/dns) for the
run-time resolver); `sockaddrFromIp` from [sockets](sockets) is the cheap way
to build a fixed destination.

## Closing

```nim
sock.close()
```

`close()` also frees the loop when the socket owns it (see above); a borrowed
loop is left alive for its owner.

## API reference

Full signatures: [UDP API](https://openpeeps.github.io/powpow/theindex.html). Related: [sockets](sockets),
[TCP](tcp).
