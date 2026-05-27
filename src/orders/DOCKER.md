# Docker Instructions — Orders Service

## Overview

The Orders service is a Spring Boot Java 21 application that manages order creation and persistence. It supports PostgreSQL and in-memory (H2) storage, and RabbitMQ, SQS, or in-memory messaging.

## Prerequisites

- Docker installed and running
- Build context must be the **repository root** (not `src/orders/`)

## Build

> ⚠️ **Important:** The `docker build` command must be run from the **root of the repository**, not from inside `src/orders/`. Running `cd src/orders && docker build .` will **fail** because the Dockerfile's `COPY` paths are relative to the repository root.

```bash
# From the root of the repository:
docker build -f src/orders/Dockerfile -t retail-store-orders .
```

## Run

### Standalone (in-memory mode — no external dependencies required)

```bash
docker run -p 8080:8080 retail-store-orders
```

The application starts with:
- **In-memory H2 database** (no PostgreSQL needed)
- **In-memory messaging** (no RabbitMQ/SQS needed)

### With PostgreSQL

```bash
docker run -p 8080:8080 \
  -e RETAIL_ORDERS_PERSISTENCE_PROVIDER=postgres \
  -e RETAIL_ORDERS_PERSISTENCE_POSTGRES_ENDPOINT=<host>:<port> \
  -e RETAIL_ORDERS_PERSISTENCE_POSTGRES_DBNAME=<dbname> \
  -e RETAIL_ORDERS_PERSISTENCE_POSTGRES_USERNAME=<username> \
  -e RETAIL_ORDERS_PERSISTENCE_POSTGRES_PASSWORD=<password> \
  retail-store-orders
```

### With RabbitMQ

```bash
docker run -p 8080:8080 \
  -e RETAIL_ORDERS_MESSAGING_PROVIDER=rabbitmq \
  -e RETAIL_ORDERS_MESSAGING_RABBITMQ_ADDRESSES=<host>:<port> \
  -e RETAIL_ORDERS_MESSAGING_RABBITMQ_USERNAME=<username> \
  -e RETAIL_ORDERS_MESSAGING_RABBITMQ_PASSWORD=<password> \
  retail-store-orders
```

### With AWS SQS

```bash
docker run -p 8080:8080 \
  -e RETAIL_ORDERS_MESSAGING_PROVIDER=sqs \
  -e RETAIL_ORDERS_MESSAGING_SQS_TOPIC=<queue-name> \
  retail-store-orders
```

## Health Check

```bash
curl http://localhost:8080/actuator/health
```

Expected response: `{"status":"UP"}`

## Environment Variables

| Variable | Description | Default |
|---|---|---|
| `RETAIL_ORDERS_PERSISTENCE_PROVIDER` | Persistence provider: `in-memory` or `postgres` | `in-memory` |
| `RETAIL_ORDERS_PERSISTENCE_POSTGRES_ENDPOINT` | PostgreSQL endpoint (`host:port`) | — |
| `RETAIL_ORDERS_PERSISTENCE_POSTGRES_DBNAME` | PostgreSQL database name | — |
| `RETAIL_ORDERS_PERSISTENCE_POSTGRES_USERNAME` | PostgreSQL username | — |
| `RETAIL_ORDERS_PERSISTENCE_POSTGRES_PASSWORD` | PostgreSQL password **(secret)** | — |
| `RETAIL_ORDERS_MESSAGING_PROVIDER` | Messaging provider: `in-memory`, `rabbitmq`, or `sqs` | `in-memory` |
| `RETAIL_ORDERS_MESSAGING_RABBITMQ_ADDRESSES` | RabbitMQ connection addresses | — |
| `RETAIL_ORDERS_MESSAGING_RABBITMQ_USERNAME` | RabbitMQ username | — |
| `RETAIL_ORDERS_MESSAGING_RABBITMQ_PASSWORD` | RabbitMQ password **(secret)** | — |
| `RETAIL_ORDERS_MESSAGING_SQS_TOPIC` | AWS SQS queue name for order events | — |
| `SPRING_PROFILES_ACTIVE` | Spring Boot active profile (e.g. `prod`) | `prod` |
| `JAVA_OPTS` | JVM options passed to the Java process | — |

## Secret Handling

For production deployments, inject secrets via:
- **ECS:** [AWS Secrets Manager integration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
- **EKS:** [Secrets Manager or KMS encryption](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html)

For AWS credentials (SQS), use IAM roles:
- **ECS:** [Task IAM Role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html) (no static keys needed)
- **EKS:** [IRSA](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)

## Exposed Port

| Port | Protocol | Description |
|---|---|---|
| `8080` | HTTP | Spring Boot REST API + Actuator endpoints |
