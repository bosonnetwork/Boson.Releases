# Boson.Releases

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

This repository is the official distribution hub for **Boson Network** binary packages. Each GitHub Release here contains the pre-built, ready-to-run artifacts produced by the [Boson](https://github.com/bosonnetwork) source tree for every supported platform and architecture.

---

## What Is Boson Network?

Boson Network is a decentralized peer-to-peer framework built on a secure Kademlia DHT. Every node and user is identified by an Ed25519 public key — no central registries, no usernames, no certificate authorities.

The platform consists of:

- **KadNode** — a Kademlia DHT for peer discovery, mutable/immutable value storage, and service registration.
- **Layer-2 Services** — WebGateway (HTTPS/Light Node API), Photon Messaging (MQTTS), Ion Store (object storage), Active Proxy (TCP NAT traversal).
- **Boson Director** — the super node supervisor managing the DHT and all services, with Client, Admin, and Federation HTTP APIs.
- **Client Libraries** — HiggsNode (Light Node), Messaging Client, Active Proxy Client.
- **`boson-director-cli`** — a native Rust CLI for the Director API.

---

## Packages

Each release provides the following artifacts:

| Platform | Architecture | Package | Type |
|---|---|---|---|
| macOS | Apple Silicon (arm64) | `boson-<v>-macos-aarch64.zip` | ZIP |
| macOS | Intel (x86_64) | `boson-<v>-macos-x86_64.zip` | ZIP |
| Linux | x86_64 | `boson-<v>-linux-x86_64.zip` | ZIP |
| Linux | x86_64 | `boson-<v>-amd64.deb` | Debian package — super node |
| Linux | x86_64 | `boson-bootstrap-<v>-amd64.deb` | Debian package — DHT bootstrap node |
| Linux | aarch64 (arm64) | `boson-<v>-linux-aarch64.zip` | ZIP |
| Linux | aarch64 (arm64) | `boson-<v>-arm64.deb` | Debian package — super node |
| Linux | aarch64 (arm64) | `boson-bootstrap-<v>-arm64.deb` | Debian package — DHT bootstrap node |
| Windows | x86_64 | `boson-<v>-windows-x86_64.zip` | ZIP |

All ZIP archives include a bundled minimal JRE built with `jlink` — no system Java installation is required.

The **bootstrap** DEB is a lightweight DHT-only package for running seed/discovery nodes without any layer-2 services.

---

## Installation

### Debian / Ubuntu (recommended for Linux servers)

Install the system dependency:

```bash
sudo apt-get install libsodium23
```

Install the super node package:

```bash
sudo dpkg -i boson-<version>-amd64.deb
```

The package installer creates a `boson` system user, generates an initial configuration, and starts the service via systemd:

```bash
sudo systemctl status boson
```

For the bootstrap (DHT-only) node:

```bash
sudo dpkg -i boson-bootstrap-<version>-amd64.deb
sudo systemctl enable boson-bootstrap
sudo systemctl start boson-bootstrap
```

### ZIP (macOS, Windows, or portable Linux)

Extract the archive and run the launcher:

```bash
unzip boson-<version>-macos-aarch64.zip
cd boson-<version>
./bin/boson.sh
```

On first run, an interactive Setup wizard starts automatically if no configuration file exists at `~/.config/boson/director.yaml`.

---

## Source

The Boson Network codebase is being progressively open-sourced:

**Already open source:**
- `boson-core` — Kademlia DHT node and DHT Shell
- Client libraries — HiggsNode, Messaging Client, Active Proxy Client

**Not yet open source** (planned):

- Layer-2 services — WebGateway, Photon Messaging, Ion Store, Active Proxy
- Boson Director — super node supervisor

---

## License

This project is licensed under the [MIT License](LICENSE).

For questions or support, contact the Boson Network team at [support@bosonnetwork.io](mailto:support@bosonnetwork.io).
