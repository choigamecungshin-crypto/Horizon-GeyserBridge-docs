# Architecture notes

## Why a proxy first?

Horizon already exposes a Java Edition dedicated server. A separate proxy can accept Bedrock's UDP/RakNet transport and create a Java TCP client session toward Horizon. This avoids coupling the first milestone to Horizon's internal gameplay APIs.

## Protocol boundary

- Bedrock side: UDP, RakNet session/reliability, Bedrock packet framing, negotiated Bedrock protocol version.
- Java side: TCP, Minecraft Java 26.3 protocol (protocol version 777), handshake/login/configuration/play states, encryption and compression as negotiated.
- Translator: stateful mapping between two different packet models. Packet IDs must never be translated by numeric substitution.

## Authentication boundary

Floodgate is not just a username rewrite. A compatible implementation must define how Bedrock identity is authenticated, how keys/signatures are verified, how the identity is conveyed to the Java server, and how spoofing is prevented. Until this is implemented and tested, the bridge must not claim Floodgate compatibility or bypass online authentication unsafely.

## Milestones

- [ ] M0: workspace, crate boundaries, CI and protocol documentation.
- [ ] M1: Java 26.3 status + login client with deterministic codec tests.
- [ ] M2: Bedrock UDP endpoint and RakNet handshake/session tests.
- [ ] M3: Bedrock login to Horizon Java login, with explicit authentication policy.
- [ ] M4: minimal playable loop (movement, keep-alive, chunks, disconnect).
- [ ] M5: chat, entities, inventory and interactions.
- [ ] M6: Floodgate-compatible identity and key verification.
- [ ] M7: real-client compatibility matrix and performance testing.

## First tests

Use packet fixtures and loopback sockets. Never require a live Mojang/Microsoft account in unit tests. End-to-end tests should use a local Horizon server and a documented Bedrock test client.
