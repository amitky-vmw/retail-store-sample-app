# ui service container

Spring Boot 3 (Java 21) storefront web UI.

## Build

The build context is the repository root. From the repository root run:

```
docker build -f src/ui/Dockerfile -t retail-ui .
```

Warning: `cd src/ui && docker build .` fails, because the COPY paths are relative to the repository root.

## Run

```
docker run --rm -p 8080:8080 retail-ui
```

Open `http://localhost:8080`. Health check: `http://localhost:8080/actuator/health`.

## Environment variables

| Variable | Purpose | Default |
| --- | --- | --- |
| PORT | HTTP port | 8080 |
| RETAIL_UI_ENDPOINTS_CATALOG | Catalog service URL | (mock data) |
| RETAIL_UI_ENDPOINTS_CARTS | Cart service URL | (mock data) |
| RETAIL_UI_ENDPOINTS_CHECKOUT | Checkout service URL | (mock data) |
| RETAIL_UI_ENDPOINTS_ORDERS | Orders service URL | (mock data) |
| RETAIL_UI_THEME | UI theme | default |
| RETAIL_UI_PRODUCTIMAGESPATH | Local folder with product images | - |
| RETAIL_UI_CHAT_ENABLED | Enable the chat bot | false |
| RETAIL_UI_CHAT_PROVIDER | Chat provider (bedrock, openai, mock) | - |
| RETAIL_UI_CHAT_BEDROCK_REGION | Bedrock Region | - |
| RETAIL_UI_CHAT_OPENAI_APIKEY | OpenAI API key (secret) | - |
| JAVA_OPTS | Extra JVM options | - |

On ECS or EKS, inject the API key through AWS Secrets Manager.
