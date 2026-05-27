# Docker Instructions — UI Service (`src/ui`)

## Overview

The UI service is a Spring Boot 3.5.5 Java application that serves the retail store frontend using Thymeleaf templates. It acts as a gateway to the catalog, cart, checkout, and orders backend services.

## Important: Build Context

> ⚠️ **The Dockerfile is located at `src/ui/Dockerfile`, but the Docker build context MUST be the repository root** (the directory containing `src/`). This is because the `pom.xml` uses the `kiota-maven-plugin` to generate API client code from sibling service OpenAPI specs (`src/orders/openapi.yml`, `src/cart/openapi.yml`, `src/catalog/openapi.yml`, `src/checkout/openapi.yml`), which are resolved relative to the build context root.

> ⚠️ **Do NOT run `cd src/ui && docker build .`** — this will fail because the Dockerfile's `COPY` paths are relative to the repository root, not the `src/ui/` directory.

## Build

Run the following command from the **root of the repository**:

```bash
# From the root of the repository (NOT from inside src/ui/):
docker build -f src/ui/Dockerfile -t retail-store-ui .
```

## Run

```bash
docker run -p 8080:8080 retail-store-ui
```

Open the UI in your browser at: [http://localhost:8080](http://localhost:8080)

## Environment Variables

The following environment variables can be passed at runtime to configure the service:

| Variable | Description | Required | Default |
|---|---|---|---|
| `RETAIL_UI_ENDPOINTS_CATALOG` | URL for the catalog service (e.g. `http://catalog:8080`) | No | _(none — catalog features disabled)_ |
| `RETAIL_UI_ENDPOINTS_CARTS` | URL for the cart service (e.g. `http://cart:8080`) | No | _(none — cart features disabled)_ |
| `RETAIL_UI_ENDPOINTS_CHECKOUT` | URL for the checkout service (e.g. `http://checkout:8080`) | No | _(none — checkout features disabled)_ |
| `RETAIL_UI_ENDPOINTS_ORDERS` | URL for the orders service (e.g. `http://orders:8080`) | No | _(none — orders features disabled)_ |
| `RETAIL_UI_THEME` | UI theme name (e.g. `default`, `orange`) | No | `default` |
| `RETAIL_UI_PRODUCT_IMAGES_PATH` | Path to custom product images directory | No | _(uses bundled images)_ |
| `RETAIL_UI_CHAT_ENABLED` | Enable the AI chat assistant (`true`/`false`) | No | `false` |
| `RETAIL_UI_CHAT_PROVIDER` | AI chat provider (`mock`, `bedrock`, `openai`) | No | _(none)_ |
| `RETAIL_UI_CHAT_BEDROCK_REGION` | AWS region for Amazon Bedrock | No | _(none)_ |
| `RETAIL_UI_CHAT_OPENAI_BASE_URL` | Base URL for OpenAI-compatible API | No | `http://localhost:8000` |
| `RETAIL_UI_CHAT_OPENAI_API_KEY` | API key for OpenAI-compatible service | No | _(none)_ |
| `SPRING_PROFILES_ACTIVE` | Spring Boot active profile (e.g. `prod`) | No | `prod` |
| `JAVA_OPTS` | JVM options (e.g. `-XX:MaxRAMPercentage=75.0`) | No | _(none)_ |

## Run with Backend Services

```bash
docker run -p 8080:8080 \
  -e RETAIL_UI_ENDPOINTS_CATALOG=http://catalog:8080 \
  -e RETAIL_UI_ENDPOINTS_CARTS=http://cart:8080 \
  -e RETAIL_UI_ENDPOINTS_CHECKOUT=http://checkout:8080 \
  -e RETAIL_UI_ENDPOINTS_ORDERS=http://orders:8080 \
  retail-store-ui
```

## Health Check

The application exposes a Spring Boot Actuator health endpoint:

```
GET /actuator/health
```

Expected response: HTTP `200 OK`

## Secret Handling

For production deployments, sensitive environment variables (e.g. `RETAIL_UI_CHAT_OPENAI_API_KEY`) should be injected securely:

- **ECS**: Use [AWS Secrets Manager integration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html) in the task definition.
- **EKS**: Use [AWS Secrets Manager or KMS encryption](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html) via Kubernetes secrets.

## Port

| Port | Protocol | Description |
|---|---|---|
| `8080` | HTTP | Spring Boot web server |
