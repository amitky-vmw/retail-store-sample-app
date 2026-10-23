# Orders Service — Docker Instructions

## Overview

The Orders service is a Java 21 / Spring Boot 3.5.5 REST API that manages order lifecycle.
It supports pluggable persistence (in-memory H2 or PostgreSQL) and messaging (in-memory, RabbitMQ, or AWS SQS).

## Build

> ⚠️ **IMPORTANT:** The build context MUST be the **root of the extracted source archive** (the repository root), NOT the `src/orders/` subdirectory.
> The Dockerfile's `COPY` instructions reference paths like `src/orders/...` and `src/misc/...` which are relative to the repository root.

```bash
# From the root of the extracted source archive (NOT from inside src/orders/):
docker build -f src/orders/Dockerfile -t retail-store-orders .
```

> ⚠️ Running `cd src/orders && docker build .` will **fail** because the Dockerfile's COPY paths are relative to the repository root, not the Dockerfile's directory. This is a common mistake.

## Run

### Default (in-memory mode — no external dependencies required)

```bash
docker run -p 8080:8080 retail-store-orders
```

### With PostgreSQL persistence

```bash
docker run -p 8080:8080 \
  -e RETAIL_ORDERS_PERSISTENCE_PROVIDER=postgres \
  -e RETAIL_ORDERS_PERSISTENCE_POSTGRES_ENDPOINT=<postgres-host>:5432 \
  -e RETAIL_ORDERS_PERSISTENCE_POSTGRES_DBNAME=orders \
  -e RETAIL_ORDERS_PERSISTENCE_POSTGRES_USERNAME=<username> \
  -e RETAIL_ORDERS_PERSISTENCE_POSTGRES_PASSWORD=<password> \
  retail-store-orders
```

### With RabbitMQ messaging

```bash
docker run -p 8080:8080 \
  -e RETAIL_ORDERS_MESSAGING_PROVIDER=rabbitmq \
  -e RETAIL_ORDERS_MESSAGING_RABBITMQ_ADDRESSES=<rabbitmq-host>:5672 \
  -e RETAIL_ORDERS_MESSAGING_RABBITMQ_USERNAME=<username> \
  -e RETAIL_ORDERS_MESSAGING_RABBITMQ_PASSWORD=<password> \
  retail-store-orders
```

### With AWS SQS messaging

```bash
docker run -p 8080:8080 \
  -e RETAIL_ORDERS_MESSAGING_PROVIDER=sqs \
  -e RETAIL_ORDERS_MESSAGING_SQS_TOPIC=<queue-name-or-url> \
  retail-store-orders
```

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `RETAIL_ORDERS_PERSISTENCE_PROVIDER` | `in-memory` | Persistence backend: `in-memory` or `postgres` |
| `RETAIL_ORDERS_PERSISTENCE_POSTGRES_ENDPOINT` | — | PostgreSQL host:port (required when provider=postgres) |
| `RETAIL_ORDERS_PERSISTENCE_POSTGRES_DBNAME` | — | PostgreSQL database name |
| `RETAIL_ORDERS_PERSISTENCE_POSTGRES_USERNAME` | — | PostgreSQL username |
| `RETAIL_ORDERS_PERSISTENCE_POSTGRES_PASSWORD` | — | PostgreSQL password (**secret**) |
| `RETAIL_ORDERS_MESSAGING_PROVIDER` | `in-memory` | Messaging backend: `in-memory`, `rabbitmq`, or `sqs` |
| `RETAIL_ORDERS_MESSAGING_RABBITMQ_ADDRESSES` | — | RabbitMQ connection addresses |
| `RETAIL_ORDERS_MESSAGING_RABBITMQ_USERNAME` | — | RabbitMQ username |
| `RETAIL_ORDERS_MESSAGING_RABBITMQ_PASSWORD` | — | RabbitMQ password (**secret**) |
| `RETAIL_ORDERS_MESSAGING_SQS_TOPIC` | — | AWS SQS queue name or URL |
| `SPRING_PROFILES_ACTIVE` | `prod` | Active Spring profile |
| `JAVA_OPTS` | — | Additional JVM options (e.g., `-Xmx512m`) |

## Health Check

```bash
curl http://localhost:8080/actuator/health
```

Expected response: `{"status":"UP"}`

## Secret Handling

For production deployments, manage secrets using:
- **ECS:** [AWS Secrets Manager integration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
- **EKS:** [Secrets Manager or KMS encryption](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html)

Do **not** pass secrets as plain environment variables in production.

## Ports

| Port | Protocol | Description |
|---|---|---|
| `8080` | HTTP | REST API and Actuator endpoints |

## API Documentation

Swagger UI is available at: `http://localhost:8080/swagger-ui.html`
