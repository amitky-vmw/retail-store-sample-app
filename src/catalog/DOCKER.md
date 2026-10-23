# Docker Build & Run Instructions — catalog

## Overview

The `catalog` service is a Go-based REST API that serves the product catalog. It supports an in-memory database (default) or MySQL for persistence.

## Prerequisites

- Docker installed and running
- Run all commands from the **root of the extracted source archive** (the directory containing `src/`)

> ⚠️ **Important:** Do NOT run `cd src/catalog && docker build .` — the Dockerfile's `COPY` paths are relative to the repository root, not the `src/catalog/` directory. Running from inside `src/catalog/` will cause the build to fail.

## Build

```bash
# From the root of the extracted source archive (NOT from inside src/catalog/):
docker build -f src/catalog/Dockerfile -t retail-store-catalog .
```

## Run

### Default (in-memory database)

```bash
docker run -p 8080:8080 retail-store-catalog
```

### With MySQL persistence

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

| Variable | Required | Default | Description |
|---|---|---|---|
| `PORT` | No | `8080` | HTTP port the server listens on |
| `RETAIL_CATALOG_PERSISTENCE_PROVIDER` | No | `in-memory` | Persistence backend: `in-memory` or `mysql` |
| `RETAIL_CATALOG_PERSISTENCE_ENDPOINT` | No | — | MySQL endpoint (`host:port`) |
| `RETAIL_CATALOG_PERSISTENCE_DB_NAME` | No | `catalogdb` | MySQL database name |
| `RETAIL_CATALOG_PERSISTENCE_USER` | No | `catalog_user` | MySQL username |
| `RETAIL_CATALOG_PERSISTENCE_PASSWORD` | No | — | MySQL password (**secret**) |
| `RETAIL_CATALOG_PERSISTENCE_CONNECT_TIMEOUT` | No | `5` | DB connection timeout (seconds) |
| `OTEL_SERVICE_NAME` | No | — | When set, enables OpenTelemetry distributed tracing |
| `GIN_MODE` | No | `release` | Gin framework mode (`release` set in Dockerfile) |

## Secret Handling

For production deployments, manage secrets using:
- **ECS:** [AWS Secrets Manager integration with ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
- **EKS:** [Secrets Manager or KMS encryption for EKS](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html)

## Health Check

```bash
curl http://localhost:8080/health
```

Expected response: `OK` (HTTP 200)

## API Endpoints

- `GET /catalog/products` — list products
- `GET /catalog/products/:id` — get product by ID
- `GET /catalog/tags` — list tags
- `GET /catalog/size` — catalog size
- `GET /health` — health check
