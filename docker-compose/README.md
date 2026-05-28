# Homelab Docker Compose

Docker Compose stack for running a local AI and monitoring environment.

## Services

| Service | Host Port | Description |
|---|---|---|
| Ollama | 11434 | Local LLM inference engine |
| Open WebUI | 3000 | Chat UI for Ollama |
| Prometheus | 9090 | Metrics collection and storage |
| Grafana | 3001 | Metrics dashboards and visualization |

## Prerequisites

- Docker and Docker Compose installed
- (Optional) NVIDIA GPU with [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) for GPU-accelerated inference

## Usage

Start all services:

```bash
docker compose up -d
```

Stop all services:

```bash
docker compose down
```

Stop and remove all volumes (resets all data):

```bash
docker compose down -v
```

View logs:

```bash
docker compose logs -f
# or for a specific service
docker compose logs -f ollama
```

## Accessing Services

- **Open WebUI** — http://localhost:3000
- **Prometheus** — http://localhost:9090
- **Grafana** — http://localhost:3001 (default login: `admin` / `admin`)

## Configuration

### Prometheus

Scrape targets are defined in `prometheus.yml`. By default it scrapes:
- Itself (`localhost:9090`)
- Ollama (`ollama:11434`)

To add more targets, edit `prometheus.yml` and restart the Prometheus container:

```bash
docker compose restart prometheus
```

### Grafana

Prometheus is automatically provisioned as the default datasource via `grafana/provisioning/datasources/prometheus.yml` — no manual setup required after first start.

To add dashboards, place JSON dashboard files in `grafana/provisioning/dashboards/`.

## File Structure

```
docker-compose/
├── docker-compose.yml
├── prometheus.yml
└── grafana/
    └── provisioning/
        └── datasources/
            └── prometheus.yml
```

## Data Persistence

All service data is stored in named Docker volumes:

| Volume | Service |
|---|---|
| `ollama_data` | Downloaded models |
| `open_webui_data` | Chat history and settings |
| `prometheus_data` | Metrics time-series data |
| `grafana_data` | Dashboards and user config |
