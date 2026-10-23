# Build stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build

RUN dnf update -y && \
    dnf install -y nodejs20 nodejs20-npm shadow-utils && \
    dnf clean all

WORKDIR /build

# Copy package manifest (drop yarn.lock since it's Berry format, incompatible with npm)
COPY src/checkout/package.json ./

# Remove packageManager field to avoid corepack enforcement, then install with npm
RUN node -e "const p=require('./package.json'); delete p.packageManager; require('fs').writeFileSync('./package.json', JSON.stringify(p, null, 2));"

# Install all dependencies with npm (devDeps needed for nest build)
RUN npm install --legacy-peer-deps

# Copy source
COPY src/checkout/src ./src
COPY src/checkout/tsconfig.json src/checkout/tsconfig.build.json src/checkout/nest-cli.json ./

# Build the NestJS application
RUN ./node_modules/.bin/nest build

# Runtime stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

RUN dnf update -y && \
    dnf install -y nodejs20 shadow-utils && \
    dnf clean all

RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

# Copy built artifacts and production node_modules from build stage
COPY --chown=appuser:appuser --from=build /build/dist ./dist
COPY --chown=appuser:appuser --from=build /build/node_modules ./node_modules

USER appuser

EXPOSE 8080

ENTRYPOINT ["node", "dist/main.js"]
