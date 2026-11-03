# Horizon Geyser Bridge

A Rust-native Bedrock-to-Java bridge targeting **Horizon Server** (Minecraft Java Edition 26.3).

> Status: design stage. Bedrock transport, RakNet and packet translation are not implemented yet. The first integration target is Horizon **Open to LAN** (integrated server), not a separate dedicated server.

## Goal

Allow Minecraft: Bedrock Edition players to join a Horizon Server by translating the Bedrock network protocol and gameplay packets to the Java Edition protocol spoken by Horizon.

This is a standalone proxy/bridge first, not a Horizon content mod. That keeps the Bedrock UDP/RakNet endpoint separate from Horizon's Java TCP server and avoids depending on an undocumented mod hook.

## Architecture

```text
Bedrock client
    | UDP / RakNet + Bedrock protocol
    v
horizon-geyser-bridge
    | Bedrock session + packet translation
    | Java protocol 777 (26.3)
    v
Horizon Server
    | existing Java Edition server protocol
    v
world / players
```

The bridge will be split into these components:

- `crates/bedrock-transport`: UDP endpoint, RakNet reliability, fragmentation and session lifecycle.
- `crates/bedrock-protocol`: Bedrock packet codecs and version negotiation.
- `crates/java-protocol`: Java 26.3 protocol client connection and packet codecs.
- `crates/translator`: login, configuration, play-state and gameplay packet translation.
- `crates/floodgate`: Bedrock identity, authentication/key handling and Java-side identity forwarding.
- `crates/bridge`: configuration, lifecycle, logging and executable.

## Compatibility strategy

1. Establish a Java 26.3 connection to Horizon using the protocol version and login flow it already supports.
2. Implement a minimal Bedrock UDP/RakNet listener and protocol-version handshake.
3. Translate status and login before attempting world gameplay.
4. Add play-state translation incrementally: keep-alive, player movement, chat, abilities, chunks, entities, inventory and interactions.
5. Add Floodgate-compatible identity/authentication only after the basic proxy path is testable.

A complete Geyser/Floodgate replacement is a large protocol project. Compatibility will be reported per feature; this repository will not claim full support until verified with real Bedrock clients and Horizon Server.

## Horizon integration facts

Horizon is a Rust implementation of Minecraft Java Edition 26.3. Its source contains the `horizon-protocol` crate, including Java packet framing, compression, encryption and typed packets. Its dedicated server accepts Java Edition connections through the existing play-state network hub. The bridge should therefore speak Java protocol to Horizon rather than patching Horizon's gameplay engine first.

## Development

The initial milestone is a compilable Rust workspace with codec and transport tests. No Minecraft assets, decompiled Minecraft code, server jars, credentials or private keys belong in this repository.

## License

To be decided after the implementation and dependency/license review. Do not copy GeyserMC or Floodgate source into this repository without checking and preserving their applicable licenses.
