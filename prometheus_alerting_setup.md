
# Prometheus & Alertmanager Alerting Setup (InstanceDown)

This documentation captures the full process of setting up alerting with Prometheus and Alertmanager, including Docker configurations, rule files, and successful alert firing verification.

---

## 1. Directory Structure

```bash
/opt/prometheus
├── alertmanager/
│   └── alertmanager.yml
├── alert.rules.yml
├── docker-compose.yml
└── prometheus.yml
```

---

## 2. Docker Compose Configuration

```yaml
version: "3.8"

services:
  prometheus:
    container_name: prometheus
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - ./alert.rules.yml:/etc/prometheus/alert.rules.yml:ro
      - prometheus-data:/prometheus
    networks:
      - messaging-network
    healthcheck:
      test: ["CMD", "wget", "--spider", "http://localhost:9090/-/ready"]
      interval: 10s
      timeout: 5s
      retries: 3

  alertmanager:
    container_name: alertmanager
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro
    networks:
      - messaging-network

volumes:
  prometheus-data:
    driver: local

networks:
  messaging-network:
    name: messaging-network
    driver: bridge
```

---

## 3. `prometheus.yml` Configuration

```yaml
global:
  scrape_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "alertmanager:9093"

rule_files:
  - "alert.rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'qorelabs_online_node'
    static_configs:
      - targets: ['qorelabs.online']
    metrics_path: /node_exporter/metrics

  - job_name: 'qorelabs_space_node'
    static_configs:
      - targets: ['qorelabs.space']
    metrics_path: /node_exporter/metrics
```

---

## 4. `alert.rules.yml` Alerting Rule

```yaml
groups:
  - name: TestAlert
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 30 seconds."
```

---

## 5. `alertmanager.yml` Email Alert Setup

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'
  group_wait: 10s
  group_interval: 30s
  repeat_interval: 1h

receivers:
  - name: 'default'
    email_configs:
      - to: 'doluwamukuye@qorelabs.org'
        from: 'alertmanager@qorelabs.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'doluwamua@gmail.com'
        auth_password: 'lribkszukayvohpf'
```

---

## 6. Reload Prometheus

```bash
docker exec prometheus kill -HUP 1
```

---

## 7. Validation

- You visited Prometheus UI: `http://89.247.214.203:9090/alerts`
- The alert **InstanceDown** for `qorelabs.space` was shown as `FIRING`
- You received an email with the alert details

---

**Status: Alerting system working and verified**
