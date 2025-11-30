# Monitoring Stack: Prometheus, Grafana, Alertmanager, Node Exporter (Docker)

This project provides a Docker-based monitoring stack using Prometheus for metrics collection, Grafana for visualization, Alertmanager for alerting, and Node Exporter for host-level metrics.

## Prerequisites

- Docker installed on the host
- Configuration files for Prometheus and Alertmanager under `/opt` on the host:
  - `/opt/prometheus.yml`
  - `/opt/alertmanager.yml`

> Adjust the paths in the commands below if your configuration files are stored in a different location.

## 1. Create Docker network

Create a dedicated Docker network so the containers can communicate by name.

```bash
docker network create monitoring
```

This needs to be run only once per host.

## 2. Run Prometheus container

Start Prometheus and expose it on port `9090`.

```bash
docker run -d \
  --name prometheus \
  --network monitoring \
  -p 9090:9090 \
  -v /opt/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v prometheus-data:/prometheus \
  prom/prometheus:latest
```

- **Prometheus UI**: http://localhost:9090

Ensure your `prometheus.yml` includes scrape configs for Node Exporter and any additional targets you want to monitor.

## 3. Run Grafana container

Start Grafana and expose it on port `3000`.

```bash
docker run -d \
  --name grafana \
  --network monitoring \
  -p 3000:3000 \
  -v grafana-data:/var/lib/grafana \
  grafana/grafana:latest
```

- **Grafana UI**: http://localhost:3000

After logging in (default credentials: `admin` / `admin`), add Prometheus as a data source:

- **URL**: `http://prometheus:9090`

## 4. Run Alertmanager container

Start Alertmanager and expose it on port `9093`.

```bash
docker run -d \
  --name alertmanager \
  --network monitoring \
  -p 9093:9093 \
  -v /opt/alertmanager.yml:/etc/alertmanager/alertmanager.yml \
  -v alertmanager-data:/alertmanager \
  prom/alertmanager:latest
```

- **Alertmanager UI**: http://localhost:9093

Configure Prometheus to send alerts to Alertmanager by adding this to `prometheus.yml`:

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

## 5. Run Node Exporter container

Run Node Exporter to expose host metrics on port `9100`.

```bash
docker run -d \
  --name node-exporter \
  --network monitoring \
  -p 9100:9100 \
  --restart unless-stopped \
  quay.io/prometheus/node-exporter:latest \

```

Add a scrape job in `prometheus.yml` to collect Node Exporter metrics:

```yaml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

## 6. Accessing the stack

Once all containers are running, access the components:

| Service | URL | Default Port |
|---------|-----|--------------|
| Prometheus | http://localhost:9090 | 9090 |
| Grafana | http://localhost:3000 | 3000 |
| Alertmanager | http://localhost:9093 | 9093 |
| Node Exporter | http://localhost:9100/metrics | 9100 |

## 7. Managing containers

**Stop all containers:**

```bash
docker stop prometheus grafana alertmanager node-exporter
```

**Start containers:**

```bash
docker start prometheus grafana alertmanager node-exporter
```

**Remove containers:**

```bash
docker rm prometheus grafana alertmanager node-exporter
```

**Remove volumes (deletes all data):**

```bash
docker volume rm prometheus-data grafana-data alertmanager-data
```

## 8. Configuration examples

### Prometheus configuration (`/opt/prometheus.yml`)

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

### Alertmanager configuration (`/opt/alertmanager.yml`)

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'default'

receivers:
  - name: 'default'
    # Configure your notification channels here (email, Slack, etc.)
```

## 9. Notes

- Do not commit secrets (passwords, API keys, webhooks) to Git
- Use specific image tags instead of `latest` for production
- Secure the UIs with authentication and TLS for remote access
- Backup your configuration files and volumes regularly
