# Changelog

All notable changes to the Boson Network binary distribution are documented here.
This project adheres to [Semantic Versioning](https://semver.org/).

## 3.1.1 (2026-09-03)

Maintenance release focused on super node deployment: running services behind a TLS-terminating reverse proxy, name access without a DNS update per connection, and a default home page for the node.

Note: The Ion Store database schema changed in this release. An Ion Store database created by 3.1.0 will fail to start under 3.1.1 with a schema version mismatch and has to be recreated.

### Added

- Add a default super node home page, served at the root, reporting node identity, status, and hosted services.
- Add name access to Active Proxy through a reverse proxy and wildcard DNS, removing the DNS update per connection.
- Add RFC 2136 DNS UPDATE support to the Active Proxy dynamic DNS provider.
- Add an independently configurable listen interface and announced public endpoint for the Photon Messaging federation interface.
- Add service-defined extra values to service configuration, announced alongside the service endpoint.
- Add a tool to reconcile the Ion Store blob directory against its database.
- Add tunable heap size and file descriptor limits for the Boson service through `/etc/default/boson`.

### Changed

- Ship the public Boson Network bootstrap nodes as the default in the bootstrap and super node configuration templates.
- Allow a service to announce a public endpoint whose scheme differs from its own listener, so services can serve plaintext behind a TLS-terminating reverse proxy.
- Serve the super node status without authentication, so the node home page can report availability to any visitor.
- Merge the Ion Store local and cache stores into a single store separated by origin, so cached objects are never served as locally hosted content.
- Report the cause when the admin CLI reaches a Director that is not serving TLS on the configured URL.
- Redesign the User Portal and Admin Dashboard sign-in pages to match the node home page.
- Declare service dependencies and startup order for the Boson systemd service.

### Fixed

- Fix Web Gateway returning HTTP 500 instead of the resolved status when a failure carries no message.
- Fix the Web Gateway client ignoring the path prefix in the gateway URL when building requests.
- Fix Active Proxy client sessions cancelling their periodic check on a successful start, which produced duplicate and stale peer announcements.
- Fix the service base path derivation in the Ion Store client.
- Fix various stability, test, documentation, and other minor issues.

## 3.1.0 (2026-08-23)

First general availability release, concluding the Community Technical Preview series.

Note: The configuration syntax was reworked in this release and is not backward compatible with 3.0.x. Existing configuration files must be regenerated or updated before upgrading.

### Added

- Add per-node announce results, reporting which nodes accepted a value or peer and why others refused.
- Add Kademlia tuning options to node configuration, including lookup, routing table, and bootstrap parameters.
- Add a common rate limiting API across the Director and Layer-2 services, with support for per-second through per-day limits.
- Add service-wide, per-address, per-user, and concurrency-based rate limiting to Web Gateway, Ion Store, and Photon Messaging.
- Add per-API request budgets, costs, and concurrency limits to Director APIs.
- Add plan-based service quotas for Ion Store, Photon Messaging, and Active Proxy.
- Add dedicated client exceptions for gateway errors, rate limits, and channel limits, including retry hints where applicable.
- Add security considerations to the protocol specification.

### Changed

- Harden the DHT against malformed and abusive traffic with bounded packet decoding, routing-table contributions, lookup responses, and improved abuse throttling.
- Improve DHT lookup and announce tasks with bounded runtime and effort, more accurate timeout and loss tracking, and proper cancellation of outstanding calls.
- Improve bootstrap reliability with warm starts, bounded bucket fan-out, faster handling of unavailable bootstrap nodes, and reduced interference with application lookups.
- Move per-packet cryptographic operations off the event loop to improve throughput under load.
- Unify service configuration across the Director and Layer-2 services and finalize the configuration syntax.
- Rename plan features to match the corresponding service terminology.
- Retune default user limits, rate limits, and concurrency settings across the Director and services.
- Remove internal DHT instances from the public API and route all sends through the node.
- Improve Web Gateway re-announcement of persistent records with bounded concurrency and lag warnings.
- Restrict key-bearing configuration files to their owning account.
- Improve super node setup and administration, including public-host-based CLI configuration and DNS resolution overrides.
- Remove bundled logging configuration from library JARs so applications can manage their own logging.
- Improve build scripts and platform packaging for a more reliable build process.

### Fixed

- Fix RPC socket binding to the wrong protocol family, preventing IPv6 nodes from starting.
- Fix messages and payloads that could exceed a single datagram.
- Fix conditions that could permanently block bootstrapping.
- Fix routing-table maintenance and warm-start sweeps skipping entries that required refresh.
- Fix task dispatch during shutdown and a race involving the local store.
- Fix bootstrap node identity being overwritten or accessed by other accounts.
- Fix stale channel membership after invalidation or early removal notifications in the messaging client.
- Fix incorrect default values when parsing Ion Store configuration.
- Fix configuration file permissions applied by the Debian post-install script.
- Fix various stability, test, documentation, and other minor issues.

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
