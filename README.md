# PINEboxed — the server variant

The `rev` of `boxedPINE`. `boxedPINE` is the **client** — it runs in the browser
(the Commons, the rooms, the gate, the SVGs). `PINEboxed` is the **server** — it
serves that client over HTTP. Two ends of one modem; the direction toggle is in
the name.

BusyBox-like: **one file** (`pineboxed`), **one applet** (`busybox httpd`). No
build, no framework, no dependency beyond BusyBox. Free, forkable, reconstructable.

```
./pineboxed check              # can it serve? busybox httpd + a docroot — exits nonzero if not
./pineboxed serve [PORT] [DOCROOT]
./pineboxed build              # build the client (../boxedPINE), then serve its dist/
```

Defaults: `PORT=8780`, client at `../boxedPINE`, docroot `../boxedPINE/dist`.
Override with `PINE_PORT`, `PINE_CLIENT`, `PINE_DOCROOT`, or `BB` (busybox path).

## Decoupled from the representation

`pineboxed` serves **any** docroot — the circuit (an HTTP server) is independent
of the projection it carries. The default docroot is the `boxedPINE` client build,
but the server does not care what it serves. Point it at a different build, a
different commons, a thumb drive — the server is the same. The representation is
swappable; the server is the carrier.

## Can say no

`pineboxed check` is the self-test: it fails (nonzero) if `busybox httpd` is
missing or the docroot has no `index.html`. The server, like everything else
here, is a thing that can refuse.

## The family (by case and by `rev`)

- `boxedPINE` — the client (browser).
- `PINEboxed` — the server (this; `rev` of the client).
- `pinEboxed` — the Obsidian protocol vault (the case bit, `0x20`, distinguishes it).
- `boxed` — migration staging (dev → boxed).
- `unboxed` — the BusyBox playground.
