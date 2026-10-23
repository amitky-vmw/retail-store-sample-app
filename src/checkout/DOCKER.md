# Docker Build & Run — checkout

## Overview
NestJS/TypeScript checkout service. Handles cart-to-order conversion with pluggable persistence (in-memory or Redis).

## Build

> ⚠️ **Important:** The build context MUST be the **repository root**, not `src/checkout/`. Running `cd src/checkout && docker build .` will fail because the Dockerfile's `COPY` paths are relative to the repository root.

```bash
# From the root of the extracted source archive (NOT from inside src/checkout/):
docker build -f src/checkout/Dockerfile -t retail-store-checkout .
```

## Run

### Minimal (in-memory persistence, no Redis required)
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

### With orders service integration
```bash
docker run -p 8080:8080 \
  -e RETAIL_CHECKOUT_ORDERS_URL=http://<orders-host>:8080 \
  retail-store-checkout
```

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `PORT` | No | `8080` | HTTP port the service listens on |
| `RETAIL_CHECKOUT_PERSISTENCE_PROVIDER` | No | `memory` | Persistence backend: `memory` or `redis` |
| `RETAIL_CHECKOUT_PERSISTENCE_REDIS_URL` | No | — | Redis connection URL (required when provider is `redis`) |
| `RETAIL_CHECKOUT_ORDERS_URL` | No | — | URL for the orders backend service |

## Health Check
```bash
curl http://localhost:8080/health
```
Expected response: HTTP 200

## Secret Handling
For production deployments, use:
- **ECS**: [Secrets Manager integration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
- **EKS**: [Secrets Manager or KMS encryption](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html)
