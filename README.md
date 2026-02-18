# Toparius — Network Topology Manager for MSPs

Toparius discovers, maps, and monitors your network infrastructure. Built for managed service providers who need visibility across multiple client environments.

## What It Does

- **Auto-discovery** — ICMP ping sweep, SNMP queries, CDP/LLDP/ARP neighbor detection
- **Topology mapping** — Visual network maps with auto-created connections from discovery data
- **Monitoring** — SNMP polling for CPU, memory, interface stats, uptime; SNMP trap receiver; syslog collector
- **Alerting** — Threshold-based alert rules with email, Slack, Discord, Teams, webhook, and PSA ticket creation
- **Multi-tenant** — Client/site/network hierarchy with RBAC and scoped client portal
- **Integrations** — Tactical RMM, UniFi controller, HaloPSA
- **IPAM** — IP address tracking per subnet with conflict detection and utilization metrics
- **Config backup** — SSH-based device config backup with change detection and diff comparison
- **Agent-based** — Deploy lightweight agents to each site for local network discovery and monitoring

## Architecture

```
┌─────────────────────────────────────────┐
│           Toparius Server               │
│  REST API · Web UI · Alert Engine       │
│  SQLite or PostgreSQL                   │
└────────────────┬────────────────────────┘
                 │ HTTPS
        ┌────────┼────────┐
        │        │        │
   ┌────▼──┐ ┌──▼────┐ ┌─▼─────┐
   │ Agent │ │ Agent │ │ Agent │
   │Site A │ │Site B │ │Site C │
   └───────┘ └───────┘ └───────┘
```

**Server**: Go binary + embedded React frontend. Single binary, zero runtime dependencies. Hosts the web UI, REST API, alert engine, and database.

**Agent**: Lightweight Go binary deployed to each site. Discovers devices, polls SNMP metrics, and reports back to the server. Auto-updates from the server.

---

## Install

### Requirements

- **OS**: Ubuntu/Debian (amd64)
- **Server**: 1 CPU, 512 MB RAM minimum
- **Agent**: Minimal — runs on any Linux box at the site
- **Network**: Agent needs ICMP and SNMP access to target subnets; server needs to be reachable from agents on port 8080 (default)

### Option 1: `.deb` Packages (Recommended)

Download the latest `.deb` packages from [Releases](https://github.com/DiAhman/toparius-releases/releases).

**Server:**

```bash
# Download and install
wget https://github.com/DiAhman/toparius-releases/releases/latest/download/toparius-server_amd64.deb
sudo dpkg -i toparius-server_amd64.deb

# Edit config (change jwt_secret!)
sudo nano /etc/toparius/toparius-server.yaml

# Start
sudo systemctl enable --now toparius-server

# Open http://your-server:8080 and complete the setup wizard
```

**Agent:**

```bash
# Download and install
wget https://github.com/DiAhman/toparius-releases/releases/latest/download/toparius-agent_amd64.deb
sudo dpkg -i toparius-agent_amd64.deb

# Edit config (set server URL and API key from the server UI)
sudo nano /etc/toparius/toparius-agent.yaml

# Start
sudo systemctl enable --now toparius-agent
```

Or use the **one-line deploy** from the Toparius server UI — go to Agents, create an agent, and copy the install command.

### Option 2: Docker

```bash
# Pull images
docker pull ghcr.io/diahman/toparius-server:latest
docker pull ghcr.io/diahman/toparius-agent:latest

# Or use docker-compose (download from this repo)
wget https://raw.githubusercontent.com/DiAhman/toparius-releases/main/docker/docker-compose.yml
wget https://raw.githubusercontent.com/DiAhman/toparius-releases/main/docker/toparius-server.yaml
wget https://raw.githubusercontent.com/DiAhman/toparius-releases/main/docker/toparius-agent.yaml

docker compose up -d
```

> **Note**: The agent container requires `network_mode: host` and `CAP_NET_RAW` for ICMP/SNMP discovery on the local network.

---

## Configuration

### Server (`/etc/toparius/toparius-server.yaml`)

```yaml
server:
  host: "0.0.0.0"
  port: 8080

database:
  driver: "sqlite"              # or "postgres"
  dsn: "/var/lib/toparius/toparius.db"

auth:
  # CHANGE THIS — generate with: openssl rand -hex 32
  jwt_secret: "change-me-in-production"
  access_token_ttl: "15m"
  refresh_token_ttl: "168h"

security:
  rate_limit_rps: 20
  rate_limit_burst: 40

# Optional: TLS
# tls:
#   enabled: true
#   cert_file: "/etc/toparius/tls/cert.pem"
#   key_file: "/etc/toparius/tls/key.pem"

log:
  level: "info"
  format: "text"
```

Environment variables override config with the `TOPARIUS_` prefix (e.g., `TOPARIUS_SERVER_PORT=9090`).

### Agent (`/etc/toparius/toparius-agent.yaml`)

```yaml
server:
  url: "http://your-server:8080"
  api_key: ""  # from Toparius server UI

agent:
  site_id: ""  # assigned during registration
  discovery_interval: "1h"
  monitor_interval: "5m"

update:
  enabled: true

log:
  level: "info"
```

---

## Upgrading

**`.deb` packages:**

```bash
# Download the new version and install over the existing one
sudo dpkg -i toparius-server_amd64.deb
sudo systemctl restart toparius-server
```

Agents auto-update from the server when a new version is available. The server `.deb` bundles the matching agent binary.

**Docker:**

```bash
docker compose pull
docker compose up -d
```

---

## Getting Started

1. Install the server and open `http://your-server:8080`
2. Complete the setup wizard (creates admin account)
3. Create a Client > Site > Network in the hierarchy
4. Create an agent and deploy it to the site using the one-line installer
5. The agent auto-discovers devices on the network and reports back
6. View your topology map, set up monitoring thresholds, and configure alerts

---

## Support & Feedback

- **Issues**: [GitHub Issues](https://github.com/DiAhman/toparius-releases/issues)
- **Email**: contact@di-ahman.com

---

## License

Toparius is proprietary software. Copyright (c) 2024-2026 DiAhman Contracting, LLC. All rights reserved.

See [THIRD-PARTY-LICENSES](THIRD-PARTY-LICENSES) for open-source dependency attribution.
