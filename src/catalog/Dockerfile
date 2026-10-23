# Build Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

RUN dnf --setopt=install_weak_deps=False install -q -y \
    git \
    golang \
    && \
    dnf clean all

RUN mkdir -p "${GOPATH}/src" "${GOPATH}/bin" /appsrc

WORKDIR /appsrc

ENV GO111MODULE=on

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
COPY --chown=appuser:appuser src/catalog/ATTRIBUTION.md /app/LICENSES.md

ENV GIN_MODE=release

EXPOSE 8080

ENTRYPOINT ["/app/main"]
