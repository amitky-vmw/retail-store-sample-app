FROM public.ecr.aws/amazonlinux/amazonlinux:2023

RUN dnf update -y && \
    dnf install -y nodejs22 nodejs22-npm shadow-utils && \
    dnf clean all

# Install yarn classic (v1)
RUN npm install -g yarn

RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /artillery

# Copy package files and install all dependencies (artillery is in devDependencies)
COPY --chown=appuser:appuser src/load-generator/package.json ./package.json
COPY --chown=appuser:appuser src/load-generator/yarn.lock ./yarn.lock

RUN yarn install --frozen-lockfile && \
    yarn cache clean

# Copy application files
COPY --chown=appuser:appuser src/load-generator/helpers.js ./helpers.js
COPY --chown=appuser:appuser src/load-generator/scenario.yml ./scenario.yml

RUN chown -R appuser:appuser /artillery

USER appuser

CMD ["/artillery/node_modules/.bin/artillery", "run", "/artillery/scenario.yml"]
