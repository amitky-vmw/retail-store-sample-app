# Build stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS builder

RUN dnf update -y && \
    dnf install -y nodejs20 nodejs20-npm shadow-utils && \
    dnf clean all

WORKDIR /build

COPY src/load-generator/package.json ./

RUN npm install --legacy-peer-deps

# Runtime stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

RUN dnf update -y && \
    dnf install -y nodejs20 shadow-utils && \
    dnf clean all

RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /artillery

COPY --chown=appuser:appuser src/load-generator/scenario.yml ./scenario.yml
COPY --chown=appuser:appuser src/load-generator/helpers.js ./helpers.js
COPY --chown=appuser:appuser --from=builder /build/node_modules ./node_modules

ENV PATH="/artillery/node_modules/.bin:$PATH"

USER appuser

CMD ["artillery", "run", "scenario.yml"]
