# orders service container

Spring Boot 3 (Java 21) orders API.

## Build

The build context is the repository root. From the repository root run:

```
docker build -f src/orders/Dockerfile -t retail-orders .
```

Warning: `cd src/orders && docker build .` fails, because the COPY paths are relative to the repository root.

## Run

```
docker run --rm -p 8080:8080 retail-orders
```

Health check: `http://localhost:8080/actuator/health`.

## Environment variables

| Variable | Purpose | Default |
| --- | --- | --- |
| PORT | HTTP port | 8080 |
| RETAIL_ORDERS_PERSISTENCE_PROVIDER | `in-memory` or `postgres` | in-memory |
| RETAIL_ORDERS_PERSISTENCE_POSTGRES_ENDPOINT | PostgreSQL host:port | - |
| RETAIL_ORDERS_PERSISTENCE_POSTGRES_DBNAME | Database name | - |
| RETAIL_ORDERS_PERSISTENCE_POSTGRES_USERNAME | Database user | - |
| RETAIL_ORDERS_PERSISTENCE_POSTGRES_PASSWORD | Database password (secret) | - |
| RETAIL_ORDERS_MESSAGING_PROVIDER | `in-memory`, `rabbitmq` or `sqs` | in-memory |
| RETAIL_ORDERS_MESSAGING_RABBITMQ_ADDRESSES | RabbitMQ addresses | - |
| RETAIL_ORDERS_MESSAGING_RABBITMQ_USERNAME | RabbitMQ user | - |
| RETAIL_ORDERS_MESSAGING_RABBITMQ_PASSWORD | RabbitMQ password (secret) | - |
| RETAIL_ORDERS_MESSAGING_SQS_TOPIC | SNS/SQS topic | - |
| JAVA_OPTS | Extra JVM options | - |

On ECS or EKS, inject the passwords through AWS Secrets Manager.
