

```markdown
# 📊 Dockerized Monitoring Stack (Prometheus, Grafana, Alertmanager)

![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=Grafana&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

A modular, containerized observability stack designed to monitor server performance, hardware metrics, and application health. This project demonstrates a standard industry implementation of the **Prometheus ecosystem** using Docker for portability and isolation.

---

## 🏗️ Architecture & Data Flow

This stack operates on a **Pull-Based Model** where Prometheus actively scrapes metrics from targets.

1.  **Node Exporter** (Agent) runs on the host, collecting kernel-level metrics (CPU, RAM, Disk I/O) and exposing them at `/metrics`.
2.  **Prometheus** (Server) periodically connects to Node Exporter, downloads the data, and stores it in its Time-Series Database (TSDB).
3.  **Grafana** (Visualization) queries Prometheus using **PromQL** to render dashboards and graphs.
4.  **Alertmanager** (Response) receives alert triggers from Prometheus (e.g., "High CPU Load") and manages notifications (deduplication, grouping, routing).

---

## 🚀 Components

| Service | Port | Description |
| :--- | :--- | :--- |
| **Prometheus** | `9090` | The core metric collector and Time-Series Database. |
| **Grafana** | `3000` | The visualization dashboard (UI). |
| **Alertmanager** | `9093` | Handles alert routing and notification grouping. |
| **Node Exporter** | `9100` | Exposes host hardware and OS metrics. |

---

## 🛠️ Prerequisites

* **Docker** installed on the host machine.
* Basic understanding of YAML configuration files.
* **Configuration Files**: You must create the following config files on your host before running the containers (recommended path: `/opt/`):
    * `/opt/prometheus.yml`
    * `/opt/alertmanager.yml`

---

## ⚡ Quick Start Guide

Follow these steps to spin up the entire stack manually.

### 1. Create a Dedicated Network
Create a bridge network to allow containers to communicate by name (Service Discovery).
```bash
docker network create monitoring

```

### 2. Configure & Run Prometheus

This container mounts the config file and a data volume for persistence.

```bash
docker run -d \
  --name prometheus \
  --network monitoring \
  -p 9090:9090 \
  -v /opt/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v prometheus-data:/prometheus \
  prom/prometheus:latest

```

### 3. Run Grafana

The dashboarding tool. Persistent storage is used for user data and dashboards.

```bash
docker run -d \
  --name grafana \
  --network monitoring \
  -p 3000:3000 \
  -v grafana-data:/var/lib/grafana \
  grafana/grafana:latest

```

* **Default Login:** `admin` / `admin`

### 4. Run Alertmanager

Handles alerts sent by Prometheus.

```bash
docker run -d \
  --name alertmanager \
  --network monitoring \
  -p 9093:9093 \
  -v /opt/alertmanager.yml:/etc/alertmanager/alertmanager.yml \
  -v alertmanager-data:/alertmanager \
  prom/alertmanager:latest

```

### 5. Run Node Exporter

The agent that exposes system metrics.

```bash
docker run -d \
  --name node-exporter \
  --network monitoring \
  -p 9100:9100 \
  --restart unless-stopped \
  quay.io/prometheus/node-exporter:latest

```

*(Note: For deep host monitoring in production, you may need to add `--net="host"` and mount `/proc` and `/sys` volumes)*

---

## ⚙️ Configuration Examples

### `prometheus.yml`

This configures Prometheus to scrape itself and the Node Exporter.

```yaml
global:
  scrape_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

```

### Connecting Grafana to Prometheus

1. Log in to Grafana (`http://localhost:3000`).
2. Go to **Connections** > **Data Sources** > **Add data source**.
3. Select **Prometheus**.
4. **URL:** `http://prometheus:9090` (Use the container name, NOT localhost).
5. Click **Save & Test**.

---

## 📦 Managing the Stack

**Stop all services:**

```bash
docker stop prometheus grafana alertmanager node-exporter

```

**Remove all containers:**

```bash
docker rm prometheus grafana alertmanager node-exporter

```

**Clean up volumes (Warning: Deletes all data):**

```bash
docker volume rm prometheus-data grafana-data alertmanager-data

```

---

## 🔮 Future Improvements

* **Docker Compose:** Migrate individual `docker run` commands to a single `docker-compose.yml` for orchestration.
* **Security:** Implement a Reverse Proxy (Nginx) with Basic Auth and SSL.
* **Alerting Rules:** Define complex alert rules (e.g., High Memory Usage > 85%) in a separate rules file.

---

## 👤 Author

**Yash**

* [GitHub Profile](https://www.google.com/search?q=https://github.com/yash252525)

```

```
