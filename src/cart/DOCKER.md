# Cart Service — Docker Build & Run Instructions

## Overview

The Cart service is a Spring Boot Java 21 application that manages shopping cart state.
It supports two persistence backends: **in-memory** (default, no external dependencies) and **DynamoDB**.

> ⚠️ **Important:** The Docker build context must be the **root of the repository**, not the `src/cart/` directory.
> All `COPY` paths in the Dockerfile are relative to the repository root.

---

## Build

Run the following command from the **root of the repository**:

```bash
# From the root of the repository (NOT from inside src/cart/):
docker build -f src/cart/Dockerfile -t retail-store-cart .
```

> ⚠️ **Do NOT run** `cd src/cart && docker build .` — this will fail because the Dockerfile's
> `COPY` paths are relative to the repository root, not the Dockerfile's directory.

---

## Run

### Standalone (in-memory persistence — no external dependencies)

```bash
docker run -p 8080:8080 retail-store-cart
```

The service will start with in-memory cart storage. No database or AWS credentials required.

### With DynamoDB persistence

```bash
docker run -p 8080:8080 \
  -e RETAIL_CART_PERSISTENCE_PROVIDER=dynamodb \
  -e RETAIL_CART_PERSISTENCE_DYNAMODB_ENDPOINT=http://<dynamodb-host>:8000 \
  -e RETAIL_CART_PERSISTENCE_DYNAMODB_CREATE_TABLE=true \
  -e AWS_REGION=us-east-1 \
  retail-store-cart
```

> 🔐 **Secret handling:** For production deployments, inject AWS credentials via IAM roles
> (ECS Task Role or EKS IRSA) rather than static `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`
> environment variables. See:
> - [ECS Secrets Manager integration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
> - [EKS security](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html)

---

## Health Check

```bash
curl http://localhost:8080/actuator/health
```

Expected response: `{"status":"UP"}`

---

## Environment Variables

| Variable | Description | Default |
|---|---|---|
| `RETAIL_CART_PERSISTENCE_PROVIDER` | Persistence backend: `in-memory` or `dynamodb` | `in-memory` |
| `RETAIL_CART_PERSISTENCE_DYNAMODB_ENDPOINT` | DynamoDB endpoint URL | _(AWS default)_ |
| `RETAIL_CART_PERSISTENCE_DYNAMODB_CREATE_TABLE` | Auto-create DynamoDB table on startup | `false` |
| `RETAIL_CART_PERSISTENCE_DYNAMODB_TABLE_NAME` | DynamoDB table name | `Items` |
| `AWS_ACCESS_KEY_ID` | AWS access key (use IAM role in production) | _(IAM role)_ |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key (use IAM role in production) | _(IAM role)_ |
| `AWS_REGION` | AWS region for DynamoDB | _(from environment)_ |
| `SPRING_PROFILES_ACTIVE` | Spring Boot active profile | `prod` |
| `JAVA_OPTS` | JVM options (e.g. `-Xmx512m`) | _(empty)_ |

---

## Exposed Port

| Port | Protocol | Description |
|---|---|---|
| `8080` | HTTP | Spring Boot REST API + Actuator endpoints |
