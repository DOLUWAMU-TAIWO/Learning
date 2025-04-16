
# Grafana + Prometheus + Docker + Nginx Setup & Troubleshooting Log

## Problem Summary

You tried to set up Grafana with Prometheus in Docker and expose it via Nginx with subpath `/grafana/`. You faced the following issues:

- Grafana container crashing (`Exited(1)`)
- `ERR_TOO_MANY_REDIRECTS` and `404 Not Found` in the browser
- Nginx was misrouting `/grafana/` requests
- Grafana trying to serve assets from `/public/...` instead of `/grafana/public/...`
- Docker network resolution issues (`prometheus` not resolving)

---

## Grafana Docker Compose Configuration (Final Working)

```yaml
version: "3.8"

services:
  grafana:
    container_name: grafana
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - ./grafana-storage:/var/lib/grafana
    networks:
      - messaging-network
    environment:
      GF_SECURITY_ADMIN_PASSWORD: mrmiagi
      GF_SERVER_ROOT_URL: "http://89.247.214.203/grafana/"
      GF_SERVER_SERVE_FROM_SUB_PATH: "true"

networks:
  messaging-network:
    driver: bridge
```

---

## Nginx Config (Final Working)

```nginx
server {
    listen 80;
    server_name 89.247.214.203;

    location /grafana/ {
        proxy_pass http://localhost:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
    }
}
```

---

## Docker Network Commands

### 1. Create a shared network:

```bash
docker network create messaging-network
```

### 2. Add a container to it (example):

```bash
docker network connect messaging-network grafana
docker network connect messaging-network prometheus
```

---

## Prometheus URL in Grafana

Set your Prometheus data source URL to:

```
http://prometheus:9090
```

Because both containers are on the same network, `prometheus` resolves internally.

---

## Final Result

Your dashboard is working perfectly and reachable via:

[http://89.247.214.203/grafana/](http://89.247.214.203/grafana/)
