# Catalog Service — Docker Build & Run Instructions

## Overview

The catalog service is a Go application that serves product catalog APIs, backed by MySQL (or an in-memory SQLite database for standalone/dev use).

## Prerequisites

- Docker installed and running
- Build context must be the **repository root** (not `src/catalog/`)

## Build

> ⚠️ **Important:** The Dockerfile uses paths relative to the repository root. You MUST run `docker build` from the repository root directory.

```bash
# From the root of the repository:
docker build -f src/catalog/Dockerfile -t retail-store-catalog .
```

> ❌ **Do NOT run:** `cd src/catalog && docker build .`
> This will fail because the Dockerfile's COPY paths are relative to the repository root, not the Dockerfile's directory.

## Run

### Standalone (in-memory database — no external dependencies)

```bash
docker run -p 8080:8080 retail-store-catalog
```

The service defaults to an in-memory SQLite database when `RETAIL_CATALOG_PERSISTENCE_PROVIDER` is not set.

### With MySQL

```bash
docker run -p 8080:8080 \
  -e RETAIL_CATALOG_PERSISTENCE_PROVIDER=mysql \
  -e RETAIL_CATALOG_PERSISTENCE_ENDPOINT=<mysql-host>:3306 \
  -e RETAIL_CATALOG_PERSISTENCE_DB_NAME=catalogdb \
  -e RETAIL_CATALOG_PERSISTENCE_USER=catalog_user \
  -e RETAIL_CATALOG_PERSISTENCE_PASSWORD=<password> \
  retail-store-catalog
```

## Environment Variables

| Variable | Description | Required | Default |
|---|---|---|---|
| `PORT` | HTTP port the service listens on | No | `8080` |
| `RETAIL_CATALOG_PERSISTENCE_PROVIDER` | Database type: `in-memory` or `mysql` | No | `in-memory` |
| `RETAIL_CATALOG_PERSISTENCE_ENDPOINT` | MySQL host:port | Only if using MySQL | — |
| `RETAIL_CATALOG_PERSISTENCE_DB_NAME` | MySQL database name | No | `catalogdb` |
| `RETAIL_CATALOG_PERSISTENCE_USER` | MySQL username | No | `catalog_user` |
| `RETAIL_CATALOG_PERSISTENCE_PASSWORD` | MySQL password | Only if using MySQL | — |
| `RETAIL_CATALOG_PERSISTENCE_CONNECT_TIMEOUT` | DB connection timeout (seconds) | No | `5` |
| `RETAIL_CATALOG_PERSISTENCE_MIGRATE` | Run DB migrations on startup | No | — |
| `OTEL_SERVICE_NAME` | OpenTelemetry service name (enables tracing) | No | — |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | OpenTelemetry collector endpoint | No | — |

## Secret Handling

For production deployments, inject `RETAIL_CATALOG_PERSISTENCE_PASSWORD` as a secret using:
- **ECS:** [AWS Secrets Manager integration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
- **EKS:** [Secrets Manager or KMS encryption](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html)

## Health Check

```
GET /health
```

Returns `200 OK` when the service is healthy.

## API Endpoints

- `GET /catalog/products` — List all products
- `GET /catalog/products/:id` — Get product by ID
- `GET /catalog/tags` — List all tags
- `GET /catalog/size` — Get catalog size
- `GET /health` — Health check
- `GET /metrics` — Prometheus metrics
