---
title: HTTP client
description: "The powpow HTTP/1.1 client: blocking HttpClient, awaitable AsyncHttpClient, keep-alive pooling, Unix sockets and HTTPS."
keywords: ["powpow", "http", "client", "httpclient", "async", "pooling"]
---

# HTTP client

`proto/httpclient.nim` ships two clients with the same request surface:

- **`HttpClient`** — blocking. Owns a private event loop, returns the
  response directly, raises `HttpError` on failure.
- **`AsyncHttpClient`** — awaitable with `std/asyncdispatch`. Same pooling,
  same response type.

Runnable example:
[`examples/httpclient.nim`](https://github.com/openpeeps/powpow/blob/main/examples/httpclient.nim)
(run `examples/httpserver.nim` on :9000 first).

## Blocking requests

```nim
let client = newHttpClient()
let res = client.get("http://localhost:9000/")
echo res.getStatusCode(), " ", res.getBodyString()
let posted = client.post("http://localhost:9000/upload", "hello from powpow client")
client.close()
```

`request` is the general form (`get`/`post` are sugar):

```nim
let res = client.request(HttpPost, "http://localhost:9000/upload",
  "hello", [("Content-Type", "text/plain")])
```

Failures raise `HttpError` (bad URL, unsupported scheme, control chars in
URL/headers, connect and response errors).

## Async requests

```nim
import std/asyncdispatch

let client = newAsyncHttpClient()
proc main() {.async.} =
  let res = await client.get("http://localhost:9000/time")
  echo res.getStatusCode(), " ", res.getBodyString()
  client.close()
waitFor main()
```

## Reading a response

```nim
res.getStatusCode()     # HttpCode
res.getStatusText()     # lent string
res.getHeaders()        # HttpHeaders
res.getBody()           # seq[byte]
res.getBodyString()     # string
res.getContentLength()  # int
res.getConnectionClose()# bool
res.isOk()              # 2xx
```

Consume headers/body before issuing the next request on the same client.
Interim `100 Continue` responses are consumed automatically, so you can send
`Expect: 100-continue`.

## Keep-alive pooling

Pooling is on by default (`keepAlive = true`): connections are reused per
origin, up to `maxIdlePerHost` per host and `maxIdleTotal` overall, each
trusted for `idleTimeoutMs`. A pooled connection the server already closed
is detected and retried transparently on a fresh connection (as long as no
response bytes arrived).

```nim
let client = newHttpClient(keepAlive = true, maxIdlePerHost = 4,
  maxIdleTotal = 32, idleTimeoutMs = 30_000)
```

`close()` drains the pool and shuts down the client's private loop.

## Unix domain sockets and HTTPS

Both variants speak to Unix-socket servers (POSIX) and HTTPS via
`newClientTlsContext`:

```nim
# UDS: URL host is ignored, transport goes over the socket path
let res = client.get("http://localhost/", [], unixSocket = "/tmp/powpow_http.sock")

# HTTPS
let tlsClient = newHttpClient(tlsCtx = newClientTlsContext())
let secure = tlsClient.get("https://example.com/")
```

## Security notes

URLs and header names/values are validated (`HttpError` on control
characters or CR/LF injection). Malformed-response hardening is covered by
`tests/test_httpclient.nim` and `tests/test_httpclient_security.nim` — see
[testing](../testing.md) and [security](../security.md).

## API reference

Full signatures:
[generated API reference](https://openpeeps.github.io/powpow/theindex.html).
Related: [HTTP server](server.md), [requests](requests.md),
[static files](static-files.md).
