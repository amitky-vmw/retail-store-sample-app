# Load Generator — Docker Build & Run Instructions

## Overview

The load generator is an [Artillery](https://www.artillery.io/)-based load testing tool that simulates user traffic against the retail store UI service. It is a **batch/CLI tool** — it runs a test scenario and exits. It does not expose any HTTP ports.

## Prerequisites

- Docker installed and running
- The retail store UI service (or another target) must be accessible from the container

## Build

> ⚠️ **Important:** The build context MUST be the **root of the extracted source archive** (the repository root), NOT the `src/load-generator/` subdirectory. The Dockerfile's `COPY` paths are relative to the repository root.

```bash
# From the root of the extracted source archive (NOT from inside src/load-generator/):
docker build -f src/load-generator/Dockerfile -t retail-store-load-generator .
```

> ❌ **Do NOT run:**
> ```bash
> cd src/load-generator && docker build .
> ```
> This will fail because the Dockerfile's COPY paths are relative to the repository root, not the Dockerfile's directory.

## Run

### Basic run (with default target — will fail without a reachable target)

```bash
docker run --rm retail-store-load-generator
```

### Run with a custom target URL

Override the target URL using the `RETAIL_LOAD_GENERATOR_TARGET_URL` environment variable:

```bash
docker run --rm \
  -e RETAIL_LOAD_GENERATOR_TARGET_URL=http://your-ui-service:8080 \
  retail-store-load-generator
```

### Run with custom duration and virtual users

Use Artillery's `--overrides` flag to set duration and arrival rate:

```bash
docker run --rm \
  retail-store-load-generator \
  /artillery/node_modules/.bin/artillery run \
    --target http://your-ui-service:8080 \
    --overrides '{"config": {"phases": [{"duration": 60, "arrivalRate": 5}]}}' \
    /artillery/scenario.yml
```

### Run against a service on the host machine

```bash
docker run --rm \
  --network host \
  -e RETAIL_LOAD_GENERATOR_TARGET_URL=http://localhost:8080 \
  retail-store-load-generator
```

### Run against a service in a Docker network

```bash
docker run --rm \
  --network my-docker-network \
  -e RETAIL_LOAD_GENERATOR_TARGET_URL=http://retail-store-ui:8080 \
  retail-store-load-generator
```

## Environment Variables

| Variable | Description | Required | Default |
|---|---|---|---|
| `RETAIL_LOAD_GENERATOR_TARGET_URL` | Target URL for the Artillery load test | No | `http://localhost:8888` |

## Notes

- The load generator is a **one-shot batch job** — it runs the test scenario and exits. It does not run as a long-lived service.
- The `scenario.yml` has `duration: 0` as a placeholder. Always override with a real duration using `--overrides`.
- For Kubernetes deployments, see the `README.md` for an example of running as a Pod with an init container.
- For secret handling, use [ECS Secrets Manager](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html) or [EKS Secrets](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html) for any sensitive environment variables.
