# Build Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

RUN dnf --setopt=install_weak_deps=False install -q -y \
    git \
    golang \
    gcc \
    gcc-c++ \
    && \
    dnf clean all

RUN mkdir -p "${GOPATH}/src" "${GOPATH}/bin" /appsrc

WORKDIR /appsrc

COPY src/catalog/go.mod src/catalog/go.sum ./
RUN go mod download

COPY src/catalog/ .

RUN go build -o main main.go

# Final stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

ENV APPUSER=appuser
ENV APPUID=1000
ENV APPGID=1000

RUN dnf --setopt=install_weak_deps=False install -q -y \
    shadow-utils \
    && \
    dnf clean all

RUN useradd \
    --home "/app" \
    --create-home \
    --user-group \
    --uid "$APPUID" \
    "$APPUSER"

WORKDIR /app
USER appuser

COPY --chown=appuser:appuser --from=build-env /appsrc/main /app/

ENV GIN_MODE=release

EXPOSE 8080

ENTRYPOINT ["/app/main"]
