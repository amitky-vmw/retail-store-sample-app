# Docker Build & Run — UI Service

## Overview

The UI service is a Spring Boot 3.5.5 Java 21 application that serves the retail store frontend using Thymeleaf templates. It proxies requests to the catalog, cart, checkout, and orders backend services.

## Important: Build Context

> ⚠️ **The build context MUST be the root of the repository, NOT the `src/ui/` directory.**

The `pom.xml` uses the `kiota-maven-plugin` to generate API client code from OpenAPI specs located in sibling service directories (`src/orders/openapi.yml`, `src/cart/openapi.yml`, `src/catalog/openapi.yml`, `src/checkout/openapi.yml`). These paths are resolved relative to the build context root.

**Running `cd src/ui && docker build .` will FAIL** because the Dockerfile's `COPY` paths are relative to the repository root — not the Dockerfile's directory.

## Build

```bash
# From the root of the repository:
docker build -f src/ui/Dockerfile -t retail-store-ui .
```

## Run

```bash
docker run -p 8080:8080 retail-store-ui
```

The UI will be available at `http://localhost:8080`. Without backend services configured, it will run in standalone/mock mode.

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `RETAIL_UI_ENDPOINTS_CATALOG` | No | URL for the catalog backend service (e.g. `http://catalog:8080`) |
| `RETAIL_UI_ENDPOINTS_CARTS` | No | URL for the cart backend service (e.g. `http://cart:8080`) |
| `RETAIL_UI_ENDPOINTS_CHECKOUT` | No | URL for the checkout backend service (e.g. `http://checkout:8080`) |
| `RETAIL_UI_ENDPOINTS_ORDERS` | No | URL for the orders backend service (e.g. `http://orders:8080`) |
| `RETAIL_UI_THEME` | No | UI theme (default: `default`) |
| `RETAIL_UI_CHAT_ENABLED` | No | Enable AI chat feature (default: `false`) |
| `RETAIL_UI_CHAT_PROVIDER` | No | Chat provider: `mock`, `bedrock`, or `openai` |
| `RETAIL_UI_CHAT_OPENAI_BASE_URL` | No | Base URL for OpenAI-compatible endpoint |
| `RETAIL_UI_CHAT_OPENAI_API_KEY` | No | API key for OpenAI chat provider (**secret**) |
| `RETAIL_UI_CHAT_BEDROCK_REGION` | No | AWS region for Amazon Bedrock |
| `SPRING_PROFILES_ACTIVE` | No | Active Spring profile (default: `prod`) |

## Run with Backend Services

```bash
docker run -p 8080:8080 \
  -e RETAIL_UI_ENDPOINTS_CATALOG=http://catalog:8080 \
  -e RETAIL_UI_ENDPOINTS_CARTS=http://cart:8080 \
  -e RETAIL_UI_ENDPOINTS_CHECKOUT=http://checkout:8080 \
  -e RETAIL_UI_ENDPOINTS_ORDERS=http://orders:8080 \
  retail-store-ui
```

## Secret Handling

For production deployments, manage secrets using:
- **ECS**: [AWS Secrets Manager integration with ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
- **EKS**: [Secrets Manager or KMS encryption with EKS](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html)

## Health Check

The Spring Boot Actuator health endpoint is available at:
```
GET http://localhost:8080/actuator/health
```

## Exposed Ports

| Port | Protocol | Description |
|---|---|---|
| `8080` | HTTP | Main application port (UI + Actuator) |
