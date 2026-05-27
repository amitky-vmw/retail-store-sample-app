# Docker Instructions — checkout service

## Overview

The `checkout` service is a NestJS TypeScript application that handles cart-to-order processing. It supports in-memory persistence (default) or Redis-backed persistence.

## Build

> ⚠️ **Important:** The build context MUST be the **root of the extracted source archive**, NOT the `src/checkout/` subdirectory. The Dockerfile's `COPY` paths are relative to the archive root.

```bash
# From the root of the extracted source archive (NOT from inside src/checkout/):
docker build -f src/checkout/Dockerfile -t retail-store-checkout .
```

> ⚠️ Running `cd src/checkout && docker build .` will **fail** because the Dockerfile's COPY paths are relative to the archive root — not the Dockerfile's directory. This is a common mistake.

## Run

### Standalone (in-memory persistence — no external dependencies)

```bash
docker run -p 8080:8080 retail-store-checkout
```

### With Redis persistence

```bash
docker run -p 8080:8080 \
  -e RETAIL_CHECKOUT_PERSISTENCE_PROVIDER=redis \
  -e RETAIL_CHECKOUT_PERSISTENCE_REDIS_URL=redis://<redis-host>:6379 \
  retail-store-checkout
```

### With Orders service integration

```bash
docker run -p 8080:8080 \
  -e RETAIL_CHECKOUT_ENDPOINTS_ORDERS=http://<orders-host>:8080 \
  retail-store-checkout
```

## Health Check

```bash
curl http://localhost:8080/health
```

Expected response: `{"status":"ok",...}`

## Environment Variables

| Variable | Description | Required | Default |
|---|---|---|---|
| `RETAIL_CHECKOUT_PERSISTENCE_PROVIDER` | Persistence backend: `in-memory` or `redis` | No | `in-memory` |
| `RETAIL_CHECKOUT_PERSISTENCE_REDIS_URL` | Redis connection URL (e.g. `redis://host:6379`) | Only if provider=redis | `""` |
| `RETAIL_CHECKOUT_PERSISTENCE_REDIS_READER_URL` | Redis reader/replica URL | No | `""` |
| `RETAIL_CHECKOUT_ENDPOINTS_ORDERS` | Base URL for the orders service | No | `""` |
| `RETAIL_CHECKOUT_SHIPPING_NAME_PREFIX` | Prefix for shipping name generation | No | `""` |
| `PORT` | HTTP port to listen on | No | `8080` |

## Secrets

For production deployments, inject secrets via:
- **ECS**: [AWS Secrets Manager integration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
- **EKS**: [Kubernetes Secrets via Secrets Manager or KMS](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html)

## Exposed Ports

| Port | Protocol | Description |
|---|---|---|
| `8080` | HTTP | NestJS API (checkout, health, metrics) |

## API Endpoints

| Endpoint | Method | Description |
|---|---|---|
| `/health` | GET | Health check |
| `/checkout/:customerId` | GET | Get checkout state |
| `/checkout/:customerId/update` | POST | Update checkout |
| `/checkout/:customerId/submit` | POST | Submit order |
| `/metrics` | GET | Prometheus metrics |
| `/topology` | GET | Service topology |
