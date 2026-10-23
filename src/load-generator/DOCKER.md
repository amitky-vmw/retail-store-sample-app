# Load Generator — Docker Build & Run Instructions

## Overview

The load generator is an [Artillery](https://www.artillery.io/)-based tool that generates synthetic HTTP traffic against the retail store UI service. It is designed to be used for load testing, autoscaling, and observability testing.

> ⚠️ **Important:** The Dockerfile uses paths relative to the **repository root** as the build context — NOT the `src/load-generator/` directory. Running `cd src/load-generator && docker build .` will **fail** because the `COPY` instructions reference `src/load-generator/...` paths that only exist relative to the repo root.

## Build

```bash
# From the root of the extracted source archive (NOT from inside src/load-generator/):
docker build -f src/load-generator/Dockerfile -t retail-store-load-generator .
```

## Run

The load generator targets the UI service. Override the default target URL using the Artillery `-t` flag:

```bash
# Run against a specific target URL
docker run --rm retail-store-load-generator \
  artillery run -t http://<UI_SERVICE_HOST>:8080 scenario.yml
```

### Run with custom scenario duration

```bash
docker run --rm retail-store-load-generator \
  artillery run -t http://<UI_SERVICE_HOST>:8080 --duration 60 --rate 5 scenario.yml
```

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| *(none)* | — | The target URL is passed via the Artillery `-t` flag at runtime |

## Notes

- This is a **batch/CLI tool** — it does not expose any HTTP ports
- The default `scenario.yml` has `duration: 0` as a placeholder — always override with `-t <target>` and `--duration <seconds>` when running
- For secret handling in ECS, use [AWS Secrets Manager integration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-secrets-manager.html)
- For EKS, use [Secrets Manager or KMS encryption](https://docs.aws.amazon.com/eks/latest/userguide/security-k8s.html)
