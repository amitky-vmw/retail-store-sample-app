# Stage 1: Build
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build

RUN dnf update -y && \
    dnf install -y nodejs20 nodejs20-npm shadow-utils && \
    dnf clean all

# Install corepack and enable Yarn 4 (Berry)
RUN npm install -g corepack && corepack enable

WORKDIR /build

# Copy package manifests and yarn files first for layer caching
COPY src/checkout/package.json src/checkout/yarn.lock src/checkout/.yarnrc.yml ./
COPY src/checkout/.yarn ./.yarn

RUN yarn install --immutable

# Copy source and build
COPY src/checkout/src ./src
COPY src/checkout/tsconfig.json src/checkout/tsconfig.build.json src/checkout/nest-cli.json ./

RUN yarn build

# Stage 2: Runtime
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

RUN dnf update -y && \
    dnf install -y nodejs20 shadow-utils && \
    dnf clean all

RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

# Copy built artifacts and production node_modules from build stage
COPY --from=build /build/dist ./dist
COPY --from=build /build/node_modules ./node_modules

RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 8080

ENTRYPOINT ["node", "dist/main.js"]
