# catalog service container

Go (Gin) product catalog API.

## Build

The build context is the repository root. From the repository root run:

```
docker build -f src/catalog/Dockerfile -t retail-catalog .
```

Warning: `cd src/catalog && docker build .` fails, because the COPY paths are relative to the repository root.

## Run

```
docker run --rm -p 8080:8080 retail-catalog
```

Health check: `http://localhost:8080/health`.

## Environment variables

| Variable | Purpose | Default |
| --- | --- | --- |
| PORT | HTTP port | 8080 |
| RETAIL_CATALOG_PERSISTENCE_PROVIDER | `in-memory` or `mysql` | in-memory |
| RETAIL_CATALOG_PERSISTENCE_ENDPOINT | MySQL/MariaDB host:port | - |
| RETAIL_CATALOG_PERSISTENCE_DB_NAME | Database name | catalogdb |
| RETAIL_CATALOG_PERSISTENCE_USER | Database user | catalog_user |
| RETAIL_CATALOG_PERSISTENCE_PASSWORD | Database password (secret) | - |
| RETAIL_CATALOG_PERSISTENCE_CONNECT_TIMEOUT | Connect timeout in seconds | 5 |
| OTEL_SERVICE_NAME | Enables OpenTelemetry tracing when set | - |

On ECS or EKS, inject the database password through AWS Secrets Manager.
