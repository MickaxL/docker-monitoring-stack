# Docker Monitoring Stack

A complete monitoring stack built with Docker Compose, featuring real-time metrics collection, visualization, and alerting. Designed as a hands-on learning project and portfolio piece demonstrating infrastructure monitoring best practices.

## Architecture
```
┌─────────────────────────────────────────────────────────┐
│                   monitoring-network                    │
│                                                         │
│  ┌──────────────┐    scrapes     ┌──────────────────┐   │
│  │ Node Exporter│◄──────────────│    Prometheus     │   │
│  │    :9100     │   every 15s   │      :9090        │   │
│  │              │               │                   │   │
│  │ CPU/RAM/Disk │               │  Time-Series DB   │   │
│  │   metrics    │               │  Alert evaluation │   │
│  └──────────────┘               └────────┬──────────┘   │
│                                     │         │         │
│                              queries│    fires│         │
│                                     │         │         │
│                                     ▼         ▼         │
│                              ┌───────────┐ ┌───────────┐│
│                              │  Grafana  │ │Alertmanager│
│                              │   :3000   │ │   :9093   ││
│                              │           │ │           ││
│                              │Dashboards │ │Grouping   ││
│                              │& Alerts   │ │& Routing  ││
│                              └───────────┘ └───────────┘│
└─────────────────────────────────────────────────────────┘
```

## Stack Components

| Component | Role | Port |
|-----------|------|------|
| **Prometheus** | Metrics collection, storage (TSDB), and alert rule evaluation | `9090` |
| **Node Exporter** | Exposes Linux system metrics (CPU, RAM, disk, network) | `9100` |
| **Grafana** | Visualization dashboards and alert management | `3000` |
| **Alertmanager** | Alert grouping, routing, and notification | `9093` |

## Features

- **Real-time system monitoring** — CPU, memory, disk, and network metrics collected every 15 seconds
- **Pre-configured alert rules** — 4 production-ready alerts:
  - `InstanceDown` — Target unreachable for 1 minute (critical)
  - `DiskSpaceOver80Percent` — Filesystem usage above 80% (warning)
  - `MemoryAvailableLow` — Available RAM below 10% (warning)
  - `HighCpuUsage` — CPU above 90% sustained for 5 minutes (warning)
- **Persistent storage** — Named volumes for Prometheus TSDB and Grafana dashboards
- **Health checks** — Every service monitored by Docker with automatic restart on failure
- **Isolated network** — Custom bridge network for service communication
- **Community dashboard** — Grafana Node Exporter Full (ID 1860) pre-imported

## Prerequisites

- Docker Engine 20.10+
- Docker Compose v2+

## Quick Start
```bash
# Clone the repository
git clone https://github.com/MickaxL/docker-monitoring-stack.git
cd docker-monitoring-stack

# Start the stack
docker compose up -d

# Verify all services are healthy
docker compose ps
```

## Access the services

| Service | URL | Credentials |
|---------|-----|-------------|
| Prometheus | http://localhost:9090 | — |
| Grafana | http://localhost:3000 | admin / admin |
| Alertmanager | http://localhost:9093 | — |
| Node Exporter | http://localhost:9100/metrics | — |

## Grafana Setup

1. Navigate to http://localhost:3000 and log in
2. Go to **Connections** → **Data sources** → **Add data source**
3. Select **Prometheus** and set URL to `http://prometheus:9090`
4. Click **Save & test**
5. Import dashboard: **Dashboards** → **New** → **Import** → Enter ID `1860` → Select Prometheus data source

## Alert Rules

All alert rules are defined in `prometheus/alert-rules.yml` and follow production best practices:

| Alert | Condition | Duration | Severity |
|-------|-----------|----------|----------|
| InstanceDown | Target unreachable (`up == 0`) | 1 min | Critical |
| DiskSpaceOver80Percent | Disk usage > 80% | 5 min | Warning |
| MemoryAvailableLow | Available RAM < 10% | 2 min | Warning |
| HighCpuUsage | CPU usage > 90% | 5 min | Warning |

Alerts can be verified at http://localhost:9090/alerts and notifications appear in Alertmanager at http://localhost:9093.

## Project Structure
```
docker-monitoring-stack/
├── docker-compose.yml              # Service orchestration
├── prometheus/
│   ├── prometheus.yml              # Scrape targets and Alertmanager connection
│   └── alert-rules.yml             # Alert rule definitions
├── alertmanager/
│   └── alertmanager.yml            # Notification routing configuration
└── README.md
```

## Technical Decisions

**Why Prometheus + Grafana?** — Industry-standard open-source monitoring stack (CNCF project). Pull-based model simplifies agent configuration. PromQL provides powerful querying for alert rules and dashboards.

**Why named volumes over bind mounts for data?** — Configuration files use bind mounts (edited locally), while data storage uses named volumes (managed by Docker, persists across restarts).

**Why custom network?** — Explicit network naming improves clarity, enables cross-stack connectivity, and follows Docker networking best practices.

**Why health checks?** — Differentiates between "process running" and "service responding." Enables automatic recovery via `restart: unless-stopped` and provides operational visibility with `docker compose ps`.

## Possible Improvements

- [ ] Add cAdvisor for container-level metrics
- [ ] Configure Alertmanager with Slack/email notifications
- [ ] Add Loki for log aggregation alongside metrics
- [ ] Deploy with Ansible for automated provisioning
- [ ] Add Nginx reverse proxy with TLS termination

## Built With

- [Prometheus](https://prometheus.io/) — Monitoring and alerting toolkit
- [Grafana](https://grafana.com/) — Observability and visualization platform
- [Node Exporter](https://github.com/prometheus/node_exporter) — System metrics exporter
- [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) — Alert notification management
- [Docker Compose](https://docs.docker.com/compose/) — Multi-container orchestration

## Author

**Mickaël Paquet** — [GitHub](https://github.com/MickaxL) | [LinkedIn](https://www.linkedin.com/in/mickael-paquet-7a0638312)