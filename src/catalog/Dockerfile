# Build stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build
RUN dnf install -y golang git gcc gcc-c++ make && \
    dnf clean all
ENV GOTOOLCHAIN=local
ENV GOPROXY=https://proxy.golang.org,direct
ENV GOFLAGS=-mod=mod
WORKDIR /build
COPY src/catalog/ ./
# CGO is required: the in-memory store uses SQLite (go-sqlite3)
RUN go version && \
    CGO_ENABLED=1 go build -o /build/main main.go

# Runtime stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023
RUN dnf install -y shadow-utils && \
    dnf clean all && \
    groupadd -r appuser && useradd -r -g appuser -d /app appuser
WORKDIR /app
COPY --chown=appuser:appuser --from=build /build/main /app/main
USER appuser
ENV GIN_MODE=release
EXPOSE 8080
ENTRYPOINT ["/app/main"]
