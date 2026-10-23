# Cart Service — Docker Instructions

## Overview

The cart service is a Spring Boot 3.5.5 / Java 21 application that provides shopping cart management APIs. It supports two persistence backends: **in-memory** (default, no external dependencies) and **Amazon DynamoDB**.

## Build

> ⚠️ **Important:** The Docker build context must be the **root of the repository**, NOT the `src/cart/` directory. The Dockerfile's `COPY` paths are relative to the repository root.

```bash
# From the root of the repository:
docker build -f src/cart/Dockerfile -t retail-store-cart .
```

> ❌ **Do NOT run** `cd src/cart && docker build .` — this will fail because the COPY paths in the Dockerfile are relative to the repository root, not the `src/cart/` directory.

## Run

### In-memory mode (default, no external dependencies)

```bash
docker run -p 8080:8080 retail-store-cart
```

### DynamoDB mode

```bash
docker run -p 8080:8080 \
  -e RETAIL_CART_PERSISTENCE_PROVIDER=dynamodb \
  -e RETAIL_CART_PERSISTENCE_DYNAMODB_ENDPOINT=http://<dynamodb-host>:8000 \
  -e RETAIL_CART_PERSISTENCE_DYNAMODB_CREATE_TABLE=true \
  -e RETAIL_CART_PERSISTENCE_DYNAMODB_TABLE_NAME=Items \
  -e AWS_ACCESS_KEY_ID=<your-access-key> \
  -e AWS_SECRET_ACCESS_KEY=<your-secret-key> \
  -e AWS_REGION=us-east-1 \
  retail-store-cart
```

## Health Check

```bash
curl http://localhost:8080/actuator/health
```

Expected response: `{"status":"UP"}`

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `RETAIL_CART_PERSISTENCE_PROVIDER` | No | `in-memory` | Persistence backend: `in-memory` or `dynamodb` |
| `RETAIL_CART_PERSISTENCE_DYNAMODB_ENDPOINT` | No | _(AWS default)_ | Custom DynamoDB endpoint URL |
| `RETAIL_CART_PERSISTENCE_DYNAMODB_CREATE_TABLE` | No | `false` | Auto-create DynamoDB table on startup |
| `RETAIL_CART_PERSISTENCE_DYNAMODB_TABLE_NAME` | No | `Items` | DynamoDB table name |
| `AWS_ACCESS_KEY_ID` | No* | — | AWS access key (required for DynamoDB mode) |
| `AWS_SECRET_ACCESS_KEY` | No* | — | AWS secret key (required for DynamoDB mode) |
| `AWS_REGION` | No* | — | AWS region (required for DynamoDB mode) |
| `SPRING_PROFILES_ACTIVE` | No | `prod` | Active Spring profile |
| `JAVA_OPTS` | No | — | Additional JVM options |

## Secret Handling

For production deployments, use:
- **ECS:** [AWS Secrets Manager integration for ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
- **EKS:** [Secrets Manager or KMS encryption for EKS](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html)

Do **not** pass `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` as plain environment variables in production. Use IAM roles for ECS tasks or EKS service accounts instead.

## Exposed Port

| Port | Protocol | Description |
|---|---|---|
| `8080` | HTTP | REST API and Spring Actuator endpoints |
