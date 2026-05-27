# Build Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

RUN dnf --setopt=install_weak_deps=False install -q -y \
    maven \
    java-21-amazon-corretto-headless \
    which \
    tar \
    gzip \
    findutils \
    && \
    dnf clean all

WORKDIR /build

# Copy the ui module build files
COPY src/ui/.mvn .mvn
COPY src/ui/mvnw .
COPY src/ui/pom.xml .

# Copy sibling OpenAPI specs required by the kiota-maven-plugin
COPY src/orders/openapi.yml ../orders/openapi.yml
COPY src/cart/openapi.yml ../cart/openapi.yml
COPY src/catalog/openapi.yml ../catalog/openapi.yml
COPY src/checkout/openapi.yml ../checkout/openapi.yml

# Copy misc style config required by checkstyle plugin
COPY src/misc/style ../misc/style

RUN ./mvnw dependency:go-offline -B -q

COPY src/ui/src ./src

RUN ./mvnw -DskipTests package -B -q && \
    mv /build/target/ui-0.0.1-SNAPSHOT.jar /app.jar

# Package Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    shadow-utils \
    findutils \
    && \
    dnf clean all

RUN dnf -q -y swap libcurl-minimal libcurl-full \
    && dnf -q -y swap curl-minimal curl-full

RUN groupadd -r appuser && useradd -r -g appuser -d /app -m appuser

ENV JAVA_TOOL_OPTIONS=
ENV SPRING_PROFILES_ACTIVE=prod

WORKDIR /app
USER appuser

COPY --chown=appuser:appuser src/ui/ATTRIBUTION.md ./LICENSES.md
COPY --chown=appuser:appuser --from=build-env /app.jar .

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
