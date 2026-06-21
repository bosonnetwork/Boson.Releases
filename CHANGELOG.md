# Changelog

All notable changes to the Boson Network binary distribution are documented here.
This project adheres to [Semantic Versioning](https://semver.org/).

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
