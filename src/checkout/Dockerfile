# Build stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build
RUN dnf install -y nodejs20 nodejs20-npm && \
    dnf clean all && \
    (command -v node || alternatives --install /usr/bin/node node /usr/bin/node-20 90) && \
    (command -v npm || alternatives --install /usr/bin/npm npm /usr/bin/npm-20 90) && \
    node --version && npm --version
RUN npm install -g corepack && \
    corepack enable && \
    corepack prepare yarn@4.11.0 --activate
ENV YARN_NODE_LINKER=node-modules
ENV COREPACK_ENABLE_DOWNLOAD_PROMPT=0
WORKDIR /build
COPY src/checkout/ ./
RUN P_HTTP="${HTTP_PROXY:-$http_proxy}" && P_HTTPS="${HTTPS_PROXY:-$https_proxy}" && \
    if [ -n "$P_HTTP" ]; then export YARN_HTTP_PROXY="$P_HTTP"; fi && \
    if [ -n "$P_HTTPS" ]; then export YARN_HTTPS_PROXY="$P_HTTPS"; fi && \
    yarn --version && \
    yarn install --immutable && \
    yarn build

# Runtime stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023
RUN dnf install -y nodejs20 shadow-utils && \
    dnf clean all && \
    (command -v node || alternatives --install /usr/bin/node node /usr/bin/node-20 90) && \
    groupadd -r appuser && useradd -r -g appuser -d /app appuser
WORKDIR /app
COPY --chown=appuser:appuser --from=build /build/package.json ./package.json
COPY --chown=appuser:appuser --from=build /build/node_modules ./node_modules
COPY --chown=appuser:appuser --from=build /build/dist ./dist
USER appuser
ENV NODE_ENV=production
EXPOSE 8080
ENTRYPOINT ["node", "dist/main.js"]
