# Setting up Prometheus and Grafana on Rocky Linux

## Overview

**Prometheus** is an open-source monitoring and alerting toolkit designed for reliability and scalability. It collects metrics from configured targets at specified intervals, stores them, and allows querying via its own language (PromQL).

**Grafana** is an open-source analytics and visualization tool. It connects to various data sources, including Prometheus, and provides customizable dashboards to visualize and monitor metrics.

In this setup, Prometheus monitors both itself and the host machine using **node_exporter**, and Grafana visualizes the collected metrics.

---

## Steps

### 1. Create Prometheus Configuration

```bash
mkdir /root/prometheus
nano /root/prometheus/prometheus.yml
```

**prometheus.yml**:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

### 2. Run Prometheus in Docker

```bash
docker run -d   --name=prometheus   -p 9090:9090   -v "/root/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml"   prom/prometheus   --config.file=/etc/prometheus/prometheus.yml
```

Access Prometheus at: `http://<VM-IP>:9090`

---

### 3. Install Node Exporter

Download and install:
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
tar xvf node_exporter-1.9.1.linux-amd64.tar.gz
cd node_exporter-1.9.1.linux-amd64
cp node_exporter /usr/bin/
cp node_exporter /usr/local/bin/
```

Run temporarily:
```bash
node_exporter &
```

Check if running:
```bash
ps aux | grep node_exporter
curl http://localhost:9100/metrics
```

Kill process:
```bash
kill <PID>
```

---

### 4. Run Node Exporter as a Systemd Service

Create `/etc/systemd/system/node_exporter.service`:
```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/bin/node_exporter
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Start service:
```bash
systemctl daemon-reload
systemctl enable node_exporter
systemctl start node_exporter
```

Check service:
```bash
curl http://localhost:9100/metrics
systemctl status node_exporter
```

---

### 5. Configure Prometheus to Scrape Node Exporter

Create `/root/prometheus/prometheus_node.yml`:
```yaml
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["192.168.245.131:9100"]
```

Run Prometheus for Node Exporter:
```bash
docker run -d --name=promnode   -p 9091:9090   -v "/root/prometheus/prometheus_node.yml:/etc/prometheus/prometheus.yml"   -v "prometheus_datav:/prometheus"   prom/prometheus   --config.file=/etc/prometheus/prometheus.yml
```

Access at: `http://192.168.245.131:9091`

Test by creating high memory usage:
```bash
dd if=/dev/zero of=/dev/shm/memory_test bs=1G count=1
rm /dev/shm/memory_test
```

---

### 6. Install and Run Grafana

```bash
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

Access Grafana at: `http://192.168.245.131:3000`

Default credentials:
- User: `admin`
- Pass: `admin` (you’ll be prompted to change)

---

### 7. Connect Grafana to Prometheus

1. Go to **Connections → Data Sources**.
2. Click **Add new data source** → Select **Prometheus**.
3. Name: `prometheus`
4. Prometheus URL: `http://172.17.0.2:9090` (container IP + port).
5. Click **Save & Test**.

---

### 8. Create a Grafana Dashboard

1. Go to **Dashboards → New → New dashboard**.
2. Click **Add visualization** → Select `prometheus`.
3. In **Metric**, choose e.g. `node_network_speed_bytes`.
4. Add label filter `device = ens160`.
5. Click **Run queries** → Save dashboard.

If **node_exporter** stops, the dashboard will have no data for that period.

---

## Summary

You have set up:
- **Prometheus** to monitor both itself and the Rocky Linux host.
- **node_exporter** to expose system metrics.
- **Grafana** to visualize those metrics.

This setup is a basic foundation for monitoring infrastructure.
