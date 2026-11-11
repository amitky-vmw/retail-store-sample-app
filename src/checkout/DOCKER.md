# checkout service container

NestJS 11 (Node.js 20) checkout API.

## Build

The build context is the repository root. From the repository root run:

```
docker build -f src/checkout/Dockerfile -t retail-checkout .
```

Warning: `cd src/checkout && docker build .` fails, because the COPY paths are relative to the repository root.

## Run

```
docker run --rm -p 8080:8080 retail-checkout
```

Health check: `http://localhost:8080/health`.

## Environment variables

| Variable | Purpose | Default |
| --- | --- | --- |
| PORT | HTTP port | 8080 |
| RETAIL_CHECKOUT_PERSISTENCE_PROVIDER | `in-memory` or `redis` | in-memory |
| RETAIL_CHECKOUT_PERSISTENCE_REDIS_URL | Redis URL, e.g. redis://host:6379 | - |
| RETAIL_CHECKOUT_PERSISTENCE_REDIS_READER_URL | Redis reader URL | - |
| RETAIL_CHECKOUT_ENDPOINTS_ORDERS | Orders service URL | (mock) |
| RETAIL_CHECKOUT_SHIPPING_NAME_PREFIX | Prefix for shipping option names | - |
