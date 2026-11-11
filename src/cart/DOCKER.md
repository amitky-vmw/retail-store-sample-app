# cart service container

Spring Boot 3 (Java 21) shopping cart API.

## Build

The build context is the repository root. From the repository root run:

```
docker build -f src/cart/Dockerfile -t retail-cart .
```

Warning: `cd src/cart && docker build .` fails, because the COPY paths are relative to the repository root.

## Run

```
docker run --rm -p 8080:8080 retail-cart
```

Health check: `http://localhost:8080/actuator/health`.

## Environment variables

| Variable | Purpose | Default |
| --- | --- | --- |
| PORT | HTTP port | 8080 |
| RETAIL_CART_PERSISTENCE_PROVIDER | `in-memory` or `dynamodb` | in-memory |
| RETAIL_CART_PERSISTENCE_DYNAMODB_TABLENAME | DynamoDB table name | Items |
| RETAIL_CART_PERSISTENCE_DYNAMODB_ENDPOINT | Custom DynamoDB endpoint (for local testing only) | (AWS default) |
| RETAIL_CART_PERSISTENCE_DYNAMODB_CREATETABLE | Create the table on start | false |
| AWS_REGION | AWS Region for DynamoDB | - |
| JAVA_OPTS | Extra JVM options | - |

On ECS or EKS, use an IAM task role or IRSA for DynamoDB access rather than static AWS keys, and inject any secret through AWS Secrets Manager.
