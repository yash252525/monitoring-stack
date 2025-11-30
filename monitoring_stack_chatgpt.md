
# Monitoring Stack: Prometheus, Grafana, Alertmanager, Node Exporter (Docker)

This project provides a Docker-based monitoring stack using Prometheus for metrics collection, Grafana for visualization, Alertmanager for alerting, and Node Exporter for host-level metrics.[web:49][web:52]

## Prerequisites

- Docker installed on the host.[web:46]
- Configuration files for Prometheus and Alertmanager under `/opt` on the host (e.g. `/opt/prometheus.yml`, `/opt/alertmanager.yml`).[web:46][web:52]

> Note: Paths in the commands below assume your config files are in `/opt`. Adjust if you use different locations.

## 1. Create Docker network

Create a dedicated network so the containers can talk to each other by name.[web:49][web:58]

```bash
docker network create monitoring
```

> You only need to run this once per host.

## 2. Run Prometheus container

Start Prometheus and expose it on port `9090`.[web:46][web:58]

```bash
docker run -d   --name prometheus   --network monitoring   -p 9090:9090   -v /opt/prometheus.yml:/etc/prometheus/prometheus.yml   -v prometheus-data:/prometheus   prom/prometheus:latest
```

- Prometheus UI: [http://localhost:9090](http://localhost:9090)  

In your `prometheus.yml`, configure scrape targets (including Node Exporter) and Alertmanager as needed.[web:46][web:52]

## 3. Run Grafana container

Start Grafana and expose it on port `3000`.[web:49][web:53]

```bash
docker run -d   --name grafana   --network monitoring   -p 3000:3000   -v grafana-data:/var/lib/grafana   grafana/grafana:latest
```

- Grafana UI: [http://localhost:3000](http://localhost:3000)  

After logging in (default `admin` / `admin`, unless changed), add Prometheus as a data source:

- URL: `http://prometheus:9090` (container-to-container over the `monitoring` network).[web:49][web:52]

## 4. Run Alertmanager container

Start Alertmanager and expose it on port `9093`.[web:50][web:55]

```bash
docker run -d   --name alertmanager   --network monitoring   -p 9093:9093   -v /opt/alertmanager.yml:/etc/alertmanager/alertmanager.yml   -v alertmanager-data:/alertmanager   prom/alertmanager:latest
```

- Alertmanager UI: [http://localhost:9093](http://localhost:9093)  

In your `prometheus.yml`, configure the alerting section to point to Alertmanager:[web:55]

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

## 5. Run Node Exporter container

Run Node Exporter to expose host metrics on port `9100`.[web:49][web:52]

```bash
docker run -d   --name node-exporter   --network monitoring   -p 9100:9100   --restart unless-stopped   -v /proc:/host/proc:ro   -v /sys:/host/sys:ro   -v /:/host:ro,rslave   quay.io/prometheus/node-exporter:latest   --path.rootfs=/host
```

Update `prometheus.yml` to scrape Node Exporter:[web:52]

```yaml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

## 6. Accessing the stack

Once all containers are running:

- Prometheus: [http://localhost:9090](http://localhost:9090)  
- Grafana: [http://localhost:3000](http://localhost:3000)  
- Alertmanager: [http://localhost:9093](http://localhost:9093)  
- Node Exporter metrics: [http://localhost:9100/metrics](http://localhost:9100/metrics)[web:49][web:52]

You can create dashboards in Grafana using Prometheus as the data source and configure alert rules in Prometheus that are routed via Alertmanager.[web:49][web:50]

## 7. Managing containers

Stop containers:

```bash
docker stop prometheus grafana alertmanager node-exporter
```

Start containers again:

```bash
docker start prometheus grafana alertmanager node-exporter
```

Remove containers:

```bash
docker rm prometheus grafana alertmanager node-exporter
```

Remove volumes if you want to delete stored data:

```bash
docker volume rm prometheus-data grafana-data alertmanager-data
```

---

### Notes / Best Practices

- Paths under `/opt` must match your actual `prometheus.yml` and `alertmanager.yml` filenames.  
- Secrets (SMTP credentials, Slack webhook URLs, etc.) should **not** be baked into images or committed to Git; keep them in external config files or environment variables.
