# Boson.Releases

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Official binary distribution hub for **Boson Network**. Each [GitHub Release](https://github.com/bosonnetwork/Boson.Releases/releases) publishes signed, ready-to-run packages, installers, and checksums for all supported platforms.

**Quick links:** [Releases](https://github.com/bosonnetwork/Boson.Releases/releases) | [Changelog](CHANGELOG.md) | [Documentation](https://docs.bosonnetwork.io) | [Java SDK (Maven Central)](https://central.sonatype.com/namespace/io.bosonnetwork)

---

## Overview

Boson Network is a decentralized peer-to-peer framework built on a Kademlia DHT where every node and user is identified by an Ed25519 key pair -- no central registries, usernames, or certificate authorities.

A super node release bundles:

| Component | Role |
|---|---|
| **Boson DHT** | Kademlia DHT for peer discovery, value storage, subscriptions, and service registration |
| **Boson Director** | Super node supervisor exposing Client, Admin, and Federation HTTP APIs plus Web Portal / Admin Dashboard |
| **Layer-2 Services** | WebGateway (HTTPS/SSE Light Node API), Photon Messaging (MQTTS + federation), Ion Store (object storage), Active Proxy (NAT traversal & named sessions), TURN Relay (bundled `coturn` STUN/TURN relay + credential API) |
| **CLI Tools** | `boson-cli` (users/developers), `boson-director-cli` (operators), `boson-node` (DHT node & shell) |

---

## Choose an Installation Path

| Deployment | Best For | Method |
|---|---|---|
| [Production Super Node](#production-super-node-ubuntu--debian-with-domain) | Public super node with a domain name, TLS, PostgreSQL, nginx, and named Active Proxy sessions | One-command setup tool (`install-super-node.sh`) |
| [Standalone DEB](#debian--ubuntu-deb) | Personal/testing super node or DHT bootstrap node without a domain | `.deb` package (`boson` / `boson-bootstrap`) |
| [Portable ZIP](#macos-linux-and-windows-zip) | Local development, macOS, Windows, or non-Debian Linux | `.zip` archive |

All release downloads are signed with the project GPG key; see [Verifying Downloads](#verifying-downloads).

---

## Production Super Node (Ubuntu / Debian with Domain)

The super node setup tool provisions PostgreSQL, nginx (with `njs`), wildcard TLS, DNS integration, UFW firewall rules, and `boson-super-node`, then runs a full-stack health check:

```bash
curl -fsSL https://github.com/bosonnetwork/Boson.Releases/releases/latest/download/install-super-node.sh \
  | sudo bash
```

It verifies its own download signature, installs to `/usr/local/lib/boson-setup`, prompts for node details, displays a plan summary, and waits for confirmation.

### Prerequisites

- **OS:** Fresh **Ubuntu 24.04 / 26.04 LTS** or **Debian 12 / 13** (`amd64` or `arm64`) with `root` / `sudo`.
- **Sizing:**

  | Resource | Minimum | Recommended (Public Node) |
  |---|---|---|
  | vCPU | 2 | 4+ |
  | RAM | 2 GB (4 GB recommended minimum) | 8 GB+ |
  | Disk | 20 GB | 100 GB+ (plus storage for Ion Store) |

- **Network:** A directly reachable static public IPv4 address (1:1 NAT is supported; port-rewriting NAT breaks DHT UDP).

### Required Inbound Ports

Open these ports in your cloud/provider firewall (and host firewall, or let the setup tool configure UFW):

| Port | Protocol | Service |
|---|---|---|
| `80` | TCP | nginx (HTTP redirect & ACME challenges) |
| `443` | TCP | nginx (APIs, consoles, TURN credential API, Active Proxy sessions) |
| `9083` | TCP | Photon Messaging (MQTTS) |
| `9090` | TCP | Active Proxy control |
| `39001` | UDP | Boson DHT (cannot be proxied) |
| `3478` | UDP / TCP | TURN relay (STUN/TURN) |
| `49152-65535` | UDP | TURN relayed media |
| `53` | TCP / UDP | PowerDNS (only when using `--dns powerdns`) |

### DNS and Wildcard TLS

Pick the node's hostname (for example, `example.com` for a dedicated domain, or `node.example.com` on a shared domain; `www.<node-domain>` is reserved for the node home page) and create two **DNS-only (non-proxied)** `A` records pointing to the server's public IPv4:

```text
node.example.com.     A  203.0.113.10
*.node.example.com.   A  203.0.113.10
```

- The wildcard record (`*`) is required for Active Proxy session hostnames and `www`.
- Do not place a CDN proxy in front of these records.
- If your DNS provider does not support wildcard records, pass `--dns powerdns` and delegate the zone via NS/glue records.
- Wildcard TLS certificates are issued over DNS-01 via a **certbot DNS plugin** (recommended for unattended renewal), **PowerDNS on-host**, or **manual TXT records** (no unattended renewal).

### Setup Options & Day-2 Operations

Pass flags to the installer after `--`:

```bash
curl -fsSL https://github.com/bosonnetwork/Boson.Releases/releases/latest/download/install-super-node.sh \
  | sudo bash -s -- --dns powerdns --firewall ufw
```

| Flag | Description |
|---|---|
| `--dns <external\|powerdns>` | DNS provider mode (default: `external`) |
| `--firewall <none\|ufw>` | Host firewall management (default: `none`) |
| `--resume` | Continue a previously stopped run from the failed step |
| `--boson-version <ver>` | Install a specific Boson release |
| `--yes` | Accept confirmation prompts non-interactively |

After installation, manage the node with the installed tool:

```bash
sudo /usr/local/lib/boson-setup/setup.sh --update        # In-place upgrade to latest release
sudo /usr/local/lib/boson-setup/scripts/healthcheck.sh   # Run full-stack health check
sudo /usr/local/lib/boson-setup/setup.sh --resume        # Resume an interrupted setup run
```

**Important paths:** `/etc/boson/` (node identity keys & config -- **back this up**), `/var/lib/boson/` (node data), `/var/lib/boson-setup/` (answers, logs, update backups), `/usr/local/lib/boson-setup/README.md` (full deployment manual).

---

## Standalone Installation (Without a Domain)

Installs Boson directly without nginx, PostgreSQL, or public TLS (uses embedded SQLite and self-signed TLS; clients connect by IP/port).

### Debian / Ubuntu (`.deb`)

Packages bundle their own minimal Java runtime and require no external dependencies.

**Super Node:**

```bash
sudo apt install ./boson-<version>-$(dpkg --print-architecture).deb
sudo systemctl status boson
```

Creates the `boson` system user, generates `/etc/boson/`, starts `boson.service`, and puts `boson-cli`, `boson-director-cli`, and `boson-node` on `PATH`.

**Bootstrap Node (DHT-only):**

```bash
sudo apt install ./boson-bootstrap-<version>-$(dpkg --print-architecture).deb
sudo systemctl enable --now boson-bootstrap
```

### macOS, Linux, and Windows (`.zip`)

```bash
unzip boson-<version>-<platform>.zip
cd boson-<version>
./bin/boson.sh    # Windows: bin\boson.cmd
```

On first launch, an interactive setup wizard writes `~/.config/boson/director.yaml` and stores node data in `~/.local/share/boson`. CLI launchers (`boson-cli`, `boson-director-cli`, `boson-node`, `.sh` / `.cmd`) are in `bin/`.

---

## Upgrading

Always review [CHANGELOG.md](CHANGELOG.md) before upgrading. Database schema migrations run automatically on first startup and are **one-way** -- downgrading requires restoring a pre-upgrade backup.

### Setup-Tool-Managed Node (Recommended)

```bash
sudo /usr/local/lib/boson-setup/setup.sh --update --check   # Dry-run check (optional)
sudo /usr/local/lib/boson-setup/setup.sh --update
sudo /usr/local/lib/boson-setup/scripts/healthcheck.sh
```

`--update` upgrades the setup tool first, verifies signatures and checksums, backs up `/etc/boson` and dumps PostgreSQL when a schema migration is detected (override with `--backup-database` or `--no-backup-database`), upgrades the package, verifies health, and rolls back automatically if a pre-migration step fails. Check any post-upgrade firewall notices printed at the end (for example, opening TURN ports when upgrading from before `3.2.1`).

### Standalone DEB Installation

```bash
# 1. Stop Boson and back up configuration and data
sudo systemctl stop boson
sudo tar -czf boson-backup-pre-<version>.tar.gz /etc/boson /var/lib/boson
# If using PostgreSQL, dump the database as well:
# sudo -u postgres pg_dump -Fc boson > boson-pre-<version>.dump

# 2. Install the new package
sudo apt install ./boson-<version>-$(dpkg --print-architecture).deb

# 3. Verify service status and running version
sudo systemctl status boson
curl -sk https://localhost:9000/api/v1/client/node   # Use http:// if behind a TLS proxy
```

DEB upgrades verify that `boson.service` is stopped and automatically apply staged configuration migrations from the previously configured version. If a newly added service encounters a port conflict (such as TURN on upgrade from before `3.2.1`), its file is saved with a `.yaml.disabled` suffix so the upgrade still succeeds.

Bootstrap node upgrade:

```bash
sudo apt install ./boson-bootstrap-<version>-$(dpkg --print-architecture).deb
```

### ZIP Archive

Configuration (`~/.config/boson`) and data (`~/.local/share/boson`) live outside the archive directory. Stop the node, back up both directories, extract the new archive, run configuration migration with the previously installed version, and restart:

```bash
unzip boson-<version>-<platform>.zip
cd boson-<version>
./jre/bin/java -cp "lib/*" io.bosonnetwork.director.Setup \
  --home . --migrate --from-version <installed-version>
./bin/boson.sh
```

### Troubleshooting Failed Upgrades

Inspect service and migration logs first:

```bash
sudo journalctl -u boson -n 100 --no-pager
```

To roll back after a database migration has run, restore your pre-upgrade database dump and `/etc/boson` backup, then reinstall the previous package version.

---

## Release Artifacts & Verification

### Packages

| Artifact | Platform / Arch | Notes |
|---|---|---|
| `boson-<v>-amd64.deb` / `boson-<v>-arm64.deb` | Linux `x86_64` / `arm64` | Super node DEB (bundled JRE + `coturn`) |
| `boson-bootstrap-<v>-amd64.deb` / `arm64.deb` | Linux `x86_64` / `arm64` | Lightweight DHT-only bootstrap node DEB |
| `boson-<v>-linux-x86_64.zip` / `linux-aarch64.zip` | Linux `x86_64` / `arm64` | Super node ZIP (bundled JRE + `coturn`) |
| `boson-<v>-macos-aarch64.zip` | macOS Apple Silicon | Super node ZIP (bundled JRE + `coturn`) |
| `boson-<v>-windows-x86_64.zip` | Windows `x86_64` | Super node ZIP (bundled JRE; no `coturn`) |
| `boson-<v>-any.zip` | Any OS / Arch | Portable JAR bundle; requires Java 17+ (no bundled JRE or `coturn`, NIO transport) |
| `install-super-node.sh`, `super-node-setup.tar.gz` | Ubuntu / Debian | Production super node setup & update tool |
| `SHA256SUMS`, `SHA256SUMS.asc` | All | GPG-signed SHA-256 manifest covering all assets |

### Verifying Downloads

Every release from `3.1.2` onward is signed with the BosonNetwork release key (published on `keys.openpgp.org` and `bosonnetwork.io`):

```text
BosonNetwork <support@bosonnetwork.io>
5A01 2C93 0EED B923 B7AB  4739 A13B 99C0 C4AA 495A
```

Verify the signature and asset checksum before installing:

```bash
gpg --keyserver hkps://keys.openpgp.org --recv-keys 5A012C930EEDB923B7AB4739A13B99C0C4AA495A
gpg --verify SHA256SUMS.asc SHA256SUMS
sha256sum --check --ignore-missing SHA256SUMS
```

The super node setup tool performs this verification automatically on install and `--update`.

---

## For Developers & Contributors

### Java SDK (Maven Central)

Boson Java libraries are published under group ID `io.bosonnetwork` with `boson-dependencies` as the Maven BOM:

- **Core:** `boson-api`, `boson-dht`
- **Clients:** `boson-higgs` (WebGateway / Light Node), `boson-director-client`, `boson-messaging-client`, `boson-ion-store-client`, `boson-active-proxy-client`

See the [Java SDK documentation](https://docs.bosonnetwork.io) for dependency snippets and API guides.

### Source Repositories

Boson Network is being open-sourced progressively:

- **Core & CLI:** [Boson.Core](https://github.com/bosonnetwork/Boson.Core), [Boson.Tools](https://github.com/bosonnetwork/Boson.Tools)
- **Client Libraries:** [Boson.WebGateway.Client](https://github.com/bosonnetwork/Boson.WebGateway.Client), [Boson.Messaging.Client](https://github.com/bosonnetwork/Boson.Messaging.Client), [Boson.IonStore.Client](https://github.com/bosonnetwork/Boson.IonStore.Client), [Boson.ActiveProxy.Client](https://github.com/bosonnetwork/Boson.ActiveProxy.Client), [Boson.Director.Client](https://github.com/bosonnetwork/Boson.Director.Client)
- **Apps:** [BosonIdentity](https://github.com/bosonnetwork/BosonIdentity), [PhotonMessenger](https://github.com/bosonnetwork/PhotonMessenger)
- **Planned open-source:** Boson Director and Layer-2 services (WebGateway, Photon Messaging, Ion Store, Active Proxy, TURN Relay)

---

## License & Support

Licensed under the [MIT License](LICENSE). For questions, bug reports, or release support, contact [support@bosonnetwork.io](mailto:support@bosonnetwork.io) or open an issue on the relevant repository.
