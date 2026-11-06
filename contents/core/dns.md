---
title: Async DNS
description: "The in-loop RFC 1035 DNS resolver: no blocking getaddrinfo, hosts/resolv.conf support, TTL caching and retries."
keywords: ["powpow", "dns", "resolver", "async", "rfc1035"]
---

# Async DNS

`net/dns.nim` implements an in-loop DNS resolver (RFC 1035). Name resolution
runs entirely on the powpow event loop — no blocking `getaddrinfo`, no worker
threads, no `--threads:on` requirement. `tcp.connect` uses it automatically, so
outbound connects never stall the loop on DNS.

Status: implemented (see [`plans/async_dns.md`](https://github.com/openpeeps/powpow/blob/main/plans/async_dns.md) for the
original design). Follow-ups since landed: TCP fallback for truncated
responses, `search`/`ndots` expansion, and IPv6 nameservers with `%scope`.

## How it works

- One non-blocking UDP socket owned by the loop (plus a second one when an
  IPv6 nameserver is configured); inbound datagrams are matched to queries
  by randomized query id.
- Sends AAAA first with A fallback (or both concurrently for
  `resolveAddrBoth`); retries across nameservers with `timeout`/`attempts`
  taken from `resolv.conf` (defaults 5s / 2).
- Reads `/etc/hosts` (IP + aliases, cached) and `/etc/resolv.conf`:
  `nameserver` entries (IPv4 and IPv6 with optional `%scope`, e.g.
  `fe80::1%eth0`), `search`/`domain` + `options ndots:` (see below).
  On macOS the stub `127.0.0.1` resolv.conf works (mDNSResponder handles
  it). Falls back to `8.8.8.8` / `1.1.1.1`. `/etc/hosts` is consulted for
  the literal input name first, so hosts entries never trigger DNS I/O.
- Truncated (TC=1) UDP answers are retried over TCP (RFC 1035 §4.2.2) on a
  transient non-blocking connection to the same nameserver; a failed
  fallback spends the attempt and retries the next nameserver.
- Short names expand through the search list: names with at least `ndots`
  dots (default 1) are tried absolute first, shorter names try each search
  domain first; a trailing dot disables expansion. Candidates are chained
  until one succeeds.
- TTL-based positive cache plus a short (30s) negative cache for NXDOMAIN.
- Numeric IP literals resolve inline with zero DNS I/O.

## Resolving an address (callback style)

```nim
resolveAddr(loop, "example.com", port = 443, SOCK_STREAM,
  proc(addrs: seq[Sockaddr_storage]; err: string) =
    if err.len > 0:
      echo "resolution failed: ", err
    else:
      echo "resolved to ", addrs.len, " address(es)"))
```

The callback fires with a sequence of `Sockaddr_storage` results, or an
empty seq plus a non-empty `err`. The callback runs on the loop.

`DnsCallback = proc(addrs: seq[Sockaddr_storage]; err: string) {.closure.}`.
`resolveAddrBoth(loop, address, port, cb)` resolves both families
(IPv6-first order for Happy Eyeballs); `resolveMx(loop, domain, mcb)` and
`resolveTxt(loop, hostname, tcb)` resolve MX/TXT records (an empty result
with no error means the name exists but publishes none — for MX, fall back
to address resolution per RFC 5321 §5.1).

## Awaitable style (`asyncdispatch` Futures)

The `*Async` names return `Future`s for `await`/`waitFor` instead of taking
callbacks — same names, different shape. Failures raise `DnsError`
(`object of CatchableError`); loop-taking forms need the caller to keep
driving the loop, loop-less forms own a private loop that is pumped and
closed before the future completes:

```nim
import std/asyncdispatch

let addrs = waitFor resolveAddrAsync("example.com", 443, SOCK_STREAM)
let mxs = waitFor resolveMxAsync("example.com")
```

## Configuring the resolver

```nim
configureDns(loop, timeoutMs = 3000, attempts = 3)       # retry policy
setDnsServers(loop, [("8.8.8.8", 53), ("2001:4860:4860::8888", 53)])
setSearchDomains(loop, ["example.com"])                  # search list override
setNdots(loop, 2)                                        # absolute-first threshold
closeDns(loop)                                            # free the resolver
```

`parseResolvConfContent(text)` exposes the `resolv.conf` parser (pure
function, unit-testable) and `parseNameserverIp("fe80::1%eth0")` parses a
single nameserver address with optional `%scope`.

## Integration with TCP connect

`tcp.connect` resolves the hostname on the loop before opening the socket, and
reports DNS failure through its `onError` callback:

```nim
connect(loop, "api.example.com", 443,
  onConnect = proc(conn: Connection) = echo "connected",
  onData = proc(conn: Connection, data: openArray[byte]) = discard,
  onError = proc(err: string) = echo "connect/DNS failed: ", err)
```

## Not yet implemented (roadmap)

- UDP `sendTo` sync cache fast-path

See [`plans/roadmap.md`](https://github.com/openpeeps/powpow/blob/main/plans/roadmap.md).

## API reference

Full signatures: [DNS API](https://openpeeps.github.io/powpow/theindex.html).
