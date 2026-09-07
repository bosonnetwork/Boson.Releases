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

Two ways in, and the right one depends on whether the node has a domain name.

| | Use | Reachable as |
|---|---|---|
| [Without a domain name](#personal-or-small-super-node-without-a-domain-name) | trying Boson, a personal node, a DHT bootstrap node | an IP address and port |
| [With a domain name](#production-super-node-with-a-domain-name) | a public super node other people rely on | `https://your.domain/`, named Active Proxy sessions, a real TLS certificate |

---

### Personal or Small Super Node Without a Domain Name

The packages alone, configured by hand. The node runs and joins the DHT, but it
has no TLS certificate of its own and no name, so clients reach it by address.

#### Debian / Ubuntu (recommended for Linux servers)

Ubuntu LTS is what we test and recommend, but the `.deb` packages install on
Debian and other Debian-based distributions just as well.

Install the system dependency:

```bash
sudo apt-get install libsodium23
```

Install the super node package:

```bash
sudo dpkg -i boson-<version>-amd64.deb
```

The package installer creates a `boson` system user, generates an initial
configuration, and starts the service via systemd:

```bash
sudo systemctl status boson
```

For the bootstrap (DHT-only) node:

```bash
sudo dpkg -i boson-bootstrap-<version>-amd64.deb
sudo systemctl enable boson-bootstrap
sudo systemctl start boson-bootstrap
```

#### ZIP (macOS, Windows, or portable Linux)

Extract the archive and run the launcher:

```bash
unzip boson-<version>-macos-aarch64.zip
cd boson-<version>
./bin/boson.sh
```

On first run, an interactive Setup wizard starts automatically if no
configuration file exists at `~/.config/boson/director.yaml`.

---

### Production Super Node with a Domain Name

A public super node needs PostgreSQL, nginx, a wildcard TLS certificate, and DNS
that resolves every Active Proxy session name. One command does all of it:

```bash
curl -fsSL https://github.com/bosonnetwork/Boson.Releases/releases/latest/download/install-super-node.sh \
  | sudo bash
```

It verifies its own download, installs to `/usr/local/lib/boson-setup`, asks a
short series of questions, shows you everything it will do, and waits for you to
confirm. About twenty minutes on a fresh server, ending with a health check
across the whole stack.

#### Before you start

**A server** - a fresh **Ubuntu LTS 24.04/26.04** or **Debian 12/13**, with
`root` or `sudo`:

| | Minimum | Recommended for a busy public node |
|---|---|---|
| vCPU | 2 | 4, dedicated |
| RAM | 2 GB | 8 GB |
| Disk | 20 GB | 100 GB, plus a volume for Ion Store objects |

Below 4 GB of RAM or 20 GB of disk the setup asks you to confirm before
continuing.

It needs a **static public IPv4 address, reachable directly**. The DHT speaks
UDP on 39001 and cannot be proxied, so a NAT that rewrites source addresses will
not work; a 1:1 NAT is fine, and the setup detects it and asks.

**These ports open to the Internet:**

```
80/tcp     nginx - HTTP redirect and ACME challenges
443/tcp    nginx - APIs, consoles, and Active Proxy sessions
9083/tcp   Photon Messaging (MQTTS)
9090/tcp   Active Proxy control
39001/udp  DHT - cannot be proxied
```

If a host firewall is active the setup offers to configure or disable it.
Provider firewalls are yours to open.

**A domain you control**, with the records below already in place.

#### The name and its DNS records

Choose the name the node will be known by:

- **Domain used only for this node** - use it directly: `example.com`
- **You also run a website there** - give the node its own name:
  `node.example.com`, leaving the apex and `www` for the site

Everything else follows from it:

| Name | Serves |
|---|---|
| `node.example.com` | APIs, admin dashboard, user portal - and the endpoints announced into the DHT, `mqtts://…:9083` and `tcp://…:9090` |
| `www.node.example.com` | the node's home page. Reserved - the setup will not accept it as the node's own name |
| `*.node.example.com` | Active Proxy sessions, one name per connected device |

Two A records, both pointing at the server:

```
node.example.com.     A  203.0.113.10
*.node.example.com.   A  203.0.113.10
```

**The wildcard is required.** A session name is not known until the session
opens, so a name nobody has allocated yet must still resolve here. It also
answers `www`, which therefore needs no record of its own.

**Leave both on DNS only - no CDN proxy.** The node's own name carries the
announced endpoints and the per-client rate limiting, and a proxy in front of it
breaks both. The setup checks the name resolves to the address it detected and
stops if it does not, so a proxied record fails visibly instead of producing a
node that looks healthy and answers nothing.

If your provider cannot store a `*` record, run `--dns powerdns` instead and
delegate the zone to the server with NS and glue records at the registrar.

#### The TLS certificate

Every name under the domain must be covered, so it is a wildcard, and Let's
Encrypt issues those over DNS-01 only. The setup offers:

- **A certbot DNS plugin** (recommended) - renews unattended. Needs an API token
  for your zone; Cloudflare, Route 53, DigitalOcean, Linode, OVH, deSEC, netcup
  and seven more are supported.
- **Manual** - you add a TXT record by hand. **Cannot renew unattended.**
- **PowerDNS on this host** - answered locally, nothing to configure at a
  provider.

#### Options

Pass them after the `--` that separates them from bash's own:

```bash
curl -fsSL <url> | sudo bash -s -- --dns powerdns --firewall ufw
```

| Option | Effect |
|---|---|
| `--dns powerdns` | run PowerDNS here instead of using your provider's DNS |
| `--firewall ufw` | configure UFW for exactly the ports above |
| `--resume` | continue a stopped run, from the step that failed |
| `--boson-version VER` | install a specific release |
| `--yes` | accept the confirmations |

A failed step stops the run, names itself, and prints the tail of its output.
Fix the cause and continue with `--resume`; nothing after it has run.

#### Afterwards

The tool stays on the host:

```bash
sudo /usr/local/lib/boson-setup/scripts/healthcheck.sh   # verify at any time
sudo /usr/local/lib/boson-setup/setup.sh --resume        # continue or re-run
```

Answers, logs and backups live in `/var/lib/boson-setup/`, the node's
configuration in `/etc/boson/`. Back up `/etc/boson` - it holds the private keys
that are this node's identity on the network and cannot be regenerated.

The full deployment manual, including doing every step by hand, is
`/usr/local/lib/boson-setup/README.md`.

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
