# Spring Monitor Demo

A minimal Spring Boot 3.x demo showcasing production-grade observability with Prometheus metrics, Grafana dashboards, Loki log aggregation, and Promtail log shipping. Comes with a ready-to-run Docker Compose stack.

## Features

- Spring Boot actuator with Prometheus endpoint (`/actuator/prometheus`)
- Centralized logging written to `logs/app.log`
- Prometheus for scraping application metrics
- Grafana for visualization (dashboards for metrics and logs)
- Loki + Promtail for log aggregation
- Single `docker-compose` to run everything locally

## Tech stack

- Java 25, Spring Boot 3.5.x
- Micrometer + Prometheus registry
- Prometheus, Grafana, Loki, Promtail
- Docker Compose

## Repository layout (high level)

```
spring-monitor-demo/
├─ docker-compose.yml
├─ springboot-app.jar              # Runnable app JAR (already built)
├─ logs/                           # Local logs (mounted to containers)
│  └─ app.log
├─ prometheus/
│  └─ prometheus.yml               # Prometheus config (mounted)
├─ loki/
│  └─ local-config.yaml            # Loki config (mounted)
├─ promtail/
│  └─ config.yaml                  # Promtail config (mounted)
├─ src/
│  └─ main/resources/application.yaml
└─ pom.xml
```

## How it works

- The Spring app exposes health/info/metrics via Spring Actuator. Metrics are available at `/actuator/prometheus` thanks to the Micrometer Prometheus registry.
- Logs are written to `logs/app.log` (configured in `application.yaml`).
- Promtail tails `logs/app.log` and ships to Loki.
- Prometheus scrapes the app at port 8080 for metrics.
- Grafana connects to Prometheus (metrics) and Loki (logs) as data sources for dashboards & exploration.

## Prerequisites

- Docker Desktop (or Docker Engine) and Docker Compose
- Optional: JDK 21+ if you want to build from source (a JAR is already included)

## Quick start (Docker Compose)

1) Start the full stack:

```bash
docker compose up -d
```

2) Verify services:

- Spring Boot app: `http://localhost:8080/actuator/health`
- Metrics endpoint: `http://localhost:8080/actuator/prometheus`
- Prometheus UI: `http://localhost:9090`
- Grafana UI: `http://localhost:3000` (default admin/admin; password set to `admin` in compose)
- Loki readiness: `http://localhost:3100/ready`

3) Generate some app traffic so metrics/logs appear (hit the app in a browser or with curl).

4) In Grafana:
- Log in → Add data sources (Prometheus at `http://prometheus:9090`, Loki at `http://loki:3100`) if not pre-provisioned.
- Explore metrics (Prometheus) and logs (Loki → query `{job="promtail"}` or the label from your promtail config).

## Application configuration

File: `src/main/resources/application.yaml`

- Server port: `8080`
- Actuator exposure: `health, info, prometheus`
- Metrics tag: `application=${spring.application.name}`
- Logging file: `logs/app.log`

Key actuator endpoints:

- `GET /actuator/health`
- `GET /actuator/info`
- `GET /actuator/prometheus`

## Build from source (optional)

If you want to rebuild the JAR instead of using `springboot-app.jar` shipped in the repo:

```bash
./mvnw clean package -DskipTests
# Output JAR will be in target/. Copy or mount it as ./springboot-app.jar for compose
```

Update `docker-compose.yml` if you change the JAR name or location, or replace the existing `springboot-app.jar` in the repository root.

## Docker Compose services and ports

- App (OpenJDK image running the JAR): `8080:8080`
- Prometheus: `9090:9090`
- Loki: `3100:3100`
- Grafana: `3000:3000`
- Promtail: tails `./logs` and ships to Loki

All services are attached to the `monitoring` Docker network.

## Logs

- Application writes to `logs/app.log` (host directory)
- Promtail reads from `./logs` and forwards to Loki
- Explore logs in Grafana → Explore → Loki data source

## Metrics

- Scraped by Prometheus from `http://springboot-app:8080/actuator/prometheus`
- View in Prometheus UI or Grafana (Prometheus data source)

## Common operations

```bash
# Start
docker compose up -d

# View logs for a container (e.g., app)
docker compose logs -f springboot-app

# Stop
docker compose down

# Recreate after changes
docker compose up -d --build --force-recreate
```

## Troubleshooting

- Grafana shows no data:
  - Ensure data sources are configured: Prometheus `http://prometheus:9090`, Loki `http://loki:3100`
  - Check Prometheus targets are up in `http://localhost:9090/targets`
- No logs in Loki:
  - Confirm `logs/app.log` exists and is being written
  - Check Promtail container logs: `docker compose logs -f promtail`
- Metrics endpoint 404:
  - Confirm `micrometer-registry-prometheus` dependency exists (see `pom.xml`)
  - Ensure actuator exposure includes `prometheus` in `application.yaml`

## License

This repository is for demo/learning purposes.

***

Happy monitoring!


