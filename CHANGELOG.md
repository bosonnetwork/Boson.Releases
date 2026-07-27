# Changelog

All notable changes to the Boson Network binary distribution are documented here.
This project adheres to [Semantic Versioning](https://semver.org/).

## 3.0.2 (2026-07-27)

Community Technical Preview 3.

### Added

- Introduce a pluggable `CryptoProvider` SPI with Bouncy Castle as the default implementation, improving crypto extensibility and compatibility.
- Add libsodium-compatible `crypto_secretstream_xchacha20poly1305` support for streaming encryption and decryption.
- Add Equihash-based asymmetric proof-of-work primitives to support permissionless registration.
- Add permissionless registration proof-of-work gate in Director to provide Sybil resistance without requiring mandatory OAuth-based registration.
- Add new Vert.x-style in-memory asynchronous `ReadStream` and `WriteStream` implementations.
- Add `crypto_kdf` compatible key derivation API and expose raw key bytes support through `CryptoBox.keyBytes()`.
- Add built-in on-the-fly encryption/decryption support in the IonStore client.
- Add object integrity verification when uploading data to IonStore.
- Add messaging client readiness watchdog to automatically detect and recover unhealthy connections during connection setup.
- Add super node QR code support to simplify mobile client discovery and onboarding.
- Add public API for querying super node identity information.

### Changed

- Change self-signed TLS certificate generation from Ed25519 certificates to ECDSA certificates to ensure better compatibility.
- Improve crypto compatibility by moving certificate generation into the crypto provider layer and expanding compatibility tests between Bouncy Castle and libsodium.
- Improve node address management:
  - Support multiple address families(IPv4 and IPv6).
  - Resolve interface bindings to live addresses during startup.
  - Hide low-level network details from public APIs for improved developer experience.
- Replace the IonStore client legacy overloaded `put()` and `get()` APIs with new fluent APIs supporting different data sources and built-in stream encryption/decryption.
- Improve HTTP connection reliability in the IonStore client by configuring keep-alive timeout for pooled connections.
- Improve super node setup utility to automatically detect public IP addresses or domain names.
- Improve Director configuration templates and service defaults.
- Improve messaging client message delivery status by leveraging early acknowledgements.
- Improve MQTT session management:
  - Pin active sessions in cache while endpoints are attached.
  - Improve session takeover handling and stale endpoint cleanup.
  - Return more precise MQTT error codes for invalid client connections.
- Improve messaging persistence architecture by introducing a unified repository abstraction.
- Improve Java compatibility for client modules.

### Fixed

- Fix MQTT session takeover race conditions and stale endpoint handling.
- Fix incorrect messaging session active address handling.
- Fix unexpected crypto context closure issues.
- Fix sending failures caused by disconnected messaging clients returning unexpected exceptions.
- Fix messaging client retry behavior by returning failed futures for unrecoverable connection errors.
- Fix JSON serialization issues on Android API level 33 by replacing DTO records with regular immutable classes.
- Fix missing channel information after fresh contact synchronization.
- Fix messaging contact event callback issues.
- Fix IonStore cache expiration behavior to prevent serving expired objects beyond their remaining TTL.
- Fix avatar cache and routing issues in Director.
- Fix various stability issues, test failures, documentation issues, and minor bugs.

## 3.0.1 (2026-06-21)

Community Technical Preview 2.

### Added
- CBOR Web Token (COSE_Sign1) implementation, now used as the bearer token for Web authentication.
- New Ion Store client library.
- User Portal and Admin Dashboard Web apps for the super node, with GitHub and Google OAuth login.
- Server-side persistent store and announcement support in the WebGateway service.

### Changed
- Common API improvements ahead of freezing the public API surface.
- Applied JSpecify nullness annotations across the API surface.
- Simplified super node setup: one private key per node, with Layer-2 services using keys derived from it.
- Reworked the Layer-2 service loading mechanism to be native-image friendly.
- Aligned the HiggsNode (Light Node) with the updated WebGateway service, making it more lightweight.

### Fixed
- Stability improvements and minor bug fixes.

## 3.0.0 (2026-05-15)

Initial Community Technical Preview release.

Includes:

- **Super Node** (Boson Director) with the following built-in Layer-2 services:
  - WebGateway
  - Ion Store
  - Active Proxy
  - Photon Messaging
- **Bootstrap Node** — lightweight DHT-only seed/discovery node.
- **Client libraries:**
  - WebGateway (HiggsNode / Light Node)
  - Active Proxy
  - Photon Messaging
