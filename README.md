# PINEboxed — the server variant

The `rev` of `boxedPINE`. `boxedPINE` is the **client** — it runs in the browser
(the Commons, the rooms, the gate, the SVGs). `PINEboxed` is the **server** — it
serves that client over HTTP. Two ends of one modem; the direction toggle is in
the name.

BusyBox-like: **one file** (`pineboxed`), **one applet** (`busybox httpd`). No
build, no framework, no dependency beyond BusyBox. Free, forkable, reconstructable.

```
./pineboxed check                 # can it serve? busybox httpd + a docroot — exits nonzero if not
./pineboxed serve [PORT] [DOCROOT] # the static client (busybox httpd)
./pineboxed bridge [BRIDGE_PORT]   # the agent channel (node): /events SSE, /message POST, tag routes
./pineboxed up                     # both: the bridge (background) + the static client (foreground)
./pineboxed build                  # build the client (../boxedPINE), then serve its dist/
```

Defaults: `PORT=8780`, `BRIDGE_PORT=8743`, client at `../boxedPINE`, docroot `../boxedPINE/dist`.
Override with `PINE_PORT`, `PINE_BRIDGE_PORT`, `PINE_CLIENT`, `PINE_DOCROOT`, `AGENT_NAME`, or `BB`.

## The agent channel (the proxy's real work)

Serving static files is the small half. The proxy's real job is **relaying the agent
conversation**: `GET /events` is the server-sent stream both sides listen to, `POST
/message` is the author speaking, `POST /agent/message` is an agent speaking, `GET
/health` reports `{ok, held}`, and agents are reached by their tag in the Host or path.

The bridge lives in the client (`repl/src/agent-bridge.ts`, coupled to
`rosetta/src/agent.ts`, run by plain `node` — no npm deps, no build), and the testbed
spawns it, so it stays in `boxedPINE`. The proxy **invokes** it rather than owning a
copy — the same way it serves the client's `dist/`: PINEboxed is the carrier, boxedPINE
is what it carries. Messages are memory-only, capped; it validates and rejects malformed
input (it can say no). `up` runs the channel and the static client together — two ports,
one proxy.

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
