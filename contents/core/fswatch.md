---
title: File system watching
description: "Monitor files and directories on the powpow loop with FileWatcher (kqueue on macOS/BSD, inotify on Linux)."
keywords: ["powpow", "fswatch", "file watching", "inotify", "kqueue"]
---

# File system watching

`fswatch.nim` exposes a `FileWatcher` that monitors a file or directory for
changes, integrated with the event loop — no polling, no extra threads.
Opt-in: `import powpow/fswatch` (it is no longer re-exported by
`import powpow`).

Runnable example: [`examples/fswatch.nim`](https://github.com/openpeeps/powpow/blob/main/examples/fswatch.nim).

## Creating a watcher

```nim
import powpow
import powpow/fswatch

let watcher = newFileWatcher(loop, "/tmp/powpow-test.txt", proc(w: FileWatcher; events: set[FileSystemEvent]) =
  if fseModified in events:
    echo "modified"
  if fseDeleted in events:
    echo "deleted"))
```

## With its own loop

Omit the `loop` and the watcher creates and owns a private one — no manual
`newLoop` needed:

```nim
let watcher = newFileWatcher("/tmp/powpow-test.txt", proc(w: FileWatcher; events: set[FileSystemEvent]) =
  if fseModified in events:
    echo "modified")

watcher.run()     # blocks, driving the owned loop
```

`watcher.getLoop()` exposes the loop (owned or borrowed) when you need to
share it, e.g. to add timers alongside the watcher. From a callback, call
`watcher.stop()` (or `watcher.close()`); after `run()` returns, `close()`
frees the owned loop. `close()` called from outside `run()` frees it
immediately. A borrowed loop is never closed by the watcher.

## Events

The callback receives the watcher and the set of events that fired.
`FileSystemEvent` values:

- `fseModified` — contents changed
- `fseCreated` — file created
- `fseDeleted` — file deleted
- `fseRenamed` — file renamed
- `fseAttrib` — attributes (permissions, times) changed
- `fseLinkCount` — hard-link count changed
- `fseRevoke` — access revoked

## Closing

```nim
watcher.close()
```

`close()` also frees the loop when the watcher owns it (see above); a borrowed
loop is left alive for its owner.

## Backends

| Platform | Mechanism |
|---|---|
| macOS / BSD | `kqueue` `EVFILT_VNODE` |
| Linux | `inotify` |
| Windows | not yet implemented |

A loop-taking watcher registers its fd on that loop (cleaned up via
`addCleanup` when the loop closes); a loop-less watcher owns a private loop
instead, freed by `close()` (see above).

## API reference

Full signatures: [fswatch API](https://openpeeps.github.io/powpow/theindex.html).
