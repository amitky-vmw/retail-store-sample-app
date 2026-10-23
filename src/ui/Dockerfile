# Build Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

# We tell DNF not to install Recommends and Suggests packages, keeping our
# installed set of packages as minimal as possible.
RUN dnf --setopt=install_weak_deps=False install -q -y \
    maven \
    java-21-amazon-corretto-headless \
    which \
    tar \
    gzip \
    findutils \
    && \
    dnf clean all

VOLUME /tmp
WORKDIR /build

# Copy Maven wrapper and POM from src/ui (build context = repo root)
COPY src/ui/.mvn .mvn
COPY src/ui/mvnw .
COPY src/ui/pom.xml .

RUN chmod +x mvnw
RUN ./mvnw dependency:go-offline -B -q

# Copy ui source
COPY src/ui/src ./src

# Copy sibling OpenAPI specs needed by kiota-maven-plugin
COPY src/orders/openapi.yml ../orders/openapi.yml
COPY src/cart/openapi.yml ../cart/openapi.yml
COPY src/catalog/openapi.yml ../catalog/openapi.yml
COPY src/checkout/openapi.yml ../checkout/openapi.yml

RUN ./mvnw -DskipTests package -B -q && \
    mv /build/target/ui-0.0.1-SNAPSHOT.jar /app.jar

# Package Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

# We tell DNF not to install Recommends and Suggests packages, which are
# weak dependencies in DNF terminology, thus keeping our installed set of
# packages as minimal as possible.
RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    shadow-utils \
    findutils \
    && \
    dnf clean all

# use curl-full to use "telnet://" scheme
# https://docs.aws.amazon.com/linux/al2023/ug/curl-minimal.html
RUN dnf -q -y swap libcurl-minimal libcurl-full \
    && dnf -q -y swap curl-minimal curl-full

ENV APPUSER=appuser
ENV APPUID=1000
ENV APPGID=1000

RUN useradd \
    --home "/app" \
    --create-home \
    --user-group \
    --uid "$APPUID" \
    "$APPUSER"

ENV JAVA_TOOL_OPTIONS=
ENV SPRING_PROFILES_ACTIVE=prod

WORKDIR /app
USER appuser

COPY --chown=appuser:appuser src/ui/ATTRIBUTION.md ./ATTRIBUTION.md
COPY --chown=appuser:appuser --from=build-env /app.jar .

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
