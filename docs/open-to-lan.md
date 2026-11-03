# Open to LAN integration

## Required user experience

1. The Horizon host opens a single-player world.
2. The host chooses **Open to LAN**.
3. The bridge notices that this world is available and exposes a Bedrock UDP endpoint.
4. A Bedrock player adds the host's LAN IP and Bedrock port, then joins.
5. The bridge creates a Java protocol connection to the exact Horizon LAN session and translates packets in both directions.
6. When the host closes the LAN world, the bridge stops accepting new sessions and closes existing sessions cleanly.

The bridge must not require `horizon-server` or a separate dedicated world.

## What the Horizon source tells us

- Single-player uses an integrated server on Horizon's server thread.
- Before LAN is opened, the integrated server starts with `NetConfig.listen = false`; the host's own player uses an in-memory connection.
- Opening the world to LAN changes the network host to accept external Java connections and announces the world on the local network.
- The LAN server uses the same Java Edition protocol implementation as the dedicated server.
- When the LAN host is signed in, the integrated server uses online-mode authentication for guests; an offline host runs the LAN world in offline mode.

This means the Bedrock bridge must attach to the *published LAN listener*, not the in-memory host connection.

## Integration options

### A. Standalone bridge (first implementation)

The bridge listens on a configurable Bedrock UDP port (default 19132), discovers or is given the Horizon LAN TCP address, then translates Bedrock sessions to Java sessions.

For the first testable version, allow an explicit target:
```sh
horizon-geyser-bridge --java-target 192.168.1.20:LAN_PORT --bedrock-bind 0.0.0.0:19132
```

This is useful for protocol development, but it is not yet the final one-click experience.

### B. Automatic Open-to-LAN detection

Prefer Horizon's LAN announcement/discovery packet when it reliably identifies the host and port. The bridge should:
- listen for the Horizon/Minecraft LAN announcement on the local network;
- match the local host's published world;
- track the current TCP port and stop when the announcement disappears or the connection fails;
- expose a local status endpoint/log so the user can see whether a world is detected.

If the announcement does not expose enough information or is not emitted on every supported platform, add a small opt-in integration hook to Horizon that reports the LAN listener's actual bound address/port to the bridge. Do not guess a port or scan arbitrary local ports.

## Authentication and identity

A Bedrock player is not a normal Java client. The bridge must not pretend a Bedrock XUID is a Mojang Java account. The authentication policy must be explicit:
- offline LAN host: use the supported offline login path and a stable, collision-resistant offline identity mapping;
- online LAN host: implement and verify the exact identity/authentication handoff required by Horizon before claiming support;
- never silently disable online-mode authentication on the host.

Floodgate-compatible identity and key verification is a later milestone, not something provided by a username rewrite.

## Milestones for Open to LAN

- [ ] OLAN-0: document the integrated-server and LAN listener lifecycle (this file).
- [ ] OLAN-1: add LAN announcement discovery and tests with captured announcement packets.
- [ ] OLAN-2: connect to a published Horizon LAN listener with a normal Java protocol test client.
- [ ] OLAN-3: expose a Bedrock UDP/RakNet listener and pass status/login handshakes.
- [ ] OLAN-4: translate the minimum playable packet set.
- [ ] OLAN-5: close/reopen handling, multiple guests, disconnects and LAN lifecycle tests.
- [ ] OLAN-6: authentication policy and Floodgate-compatible identity.

## Current status

None of the Bedrock transport, RakNet, Bedrock packet translation, or automatic LAN discovery is implemented yet. The repository currently contains design documentation only. A successful Java TCP connection alone is not Bedrock compatibility.
