# Transport — discovery, first connection, then UDP

The proxy's transport is three layers, in one order: **discover → agree → carry.**
It is the XNOR-before-XOR order as a network stack — the shared ground is
established first, and only then does the fast, connectionless layer run. UDP auth
alone is meaningless (RFC 9868: stateless, fail-open); UDP *after a first
connection* is fine, because the first connection supplies the key the packets are
checked against.

## Layer 0 — discovery (who is there)

- **HTTP** — the bridge: `GET /health`, `GET /events`, tag routes (`pineboxed bridge`).
- **MQTT** — `repl/src/presence.ts`: subscribe to `omi/presence/<address>`; a retained
  message means a peer has announced, silence means none has. The peer announces
  itself retained, with a will that clears it when it drops.

Discovery answers *who shares the ground* — it does not yet establish it.

## Layer 1 — first connection (establish the shared ground)  ·  XNOR

The handshake that coordinates keys and a starting sequence. This is the "same
before the different": no agreement here, nothing downstream can be verified
(the hashing prediction proved it — data must exist to verify).

- Signaling rides **MQTT** (publish/subscribe the WebRTC offer / answer / ICE on a
  signal topic) so the bridge's authoritative grammar (`rosetta/src/agent.ts`:
  `say`/`point`/`show`) stays untouched. (A new bridge message kind is the
  alternative, but it changes the verified grammar — avoided unless needed.)

## Layer 2 — carry (the fast relay)  ·  XOR, over keyed UDP

- **WebRTC datachannel** — peer-to-peer, **DTLS over UDP**. This *is* "UDP after
  first connection," done by the standard: the ICE/DTLS handshake (set up by the
  signaling in Layer 1) coordinates the keys, then data flows over UDP, each packet
  authenticated against those keys. No hand-rolled crypto — WebRTC owns the keying.
- The proxy **introduces** the peers (Layers 0–1) and then **steps aside**: the data
  is peer-to-peer, not through the proxy. PINEboxed is the carrier that hands peers
  to each other, not a relay they depend on once connected.

## The order is enforced, not chosen

No discovery → no peer. No signaling handshake → no DTLS keys → no datachannel. A
UDP packet with no established key is dropped. So the stack cannot run out of order:
the same (the key) must exist before the different (the data) can be measured.
That is the propagation axiom as a transport.

## What exists / what is next

- **exists:** HTTP bridge (`pineboxed serve|bridge|up`), MQTT presence (`presence.ts`).
- **next (build at a browser, where WebRTC can be verified):**
  1. an MQTT signal topic + helper in `presence.ts` (publish/subscribe SDP + ICE);
  2. `RTCPeerConnection` + `RTCDataChannel` in `space/agent.html`, signalling over it;
  3. perfect-negotiation glue (offer/answer/rollback), tested against a reference peer.

Loopback note: the bridge binds `127.0.0.1`; for peers on an untrusted network the
only honest keyed-UDP is WebRTC's DTLS (or QUIC) — not a hand-rolled MAC.
