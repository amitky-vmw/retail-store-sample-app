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

# Copy Maven wrapper and POM from repo root context
COPY src/orders/.mvn .mvn
COPY src/orders/mvnw .
COPY src/orders/pom.xml .

RUN chmod +x mvnw

# Copy checkstyle config referenced as ../misc/style/java/checkstyle.xml from src/orders/
COPY src/misc/style/java/checkstyle.xml ../misc/style/java/checkstyle.xml

RUN ./mvnw dependency:go-offline -B -q

# Copy application source
COPY src/orders/src ./src

RUN ./mvnw -DskipTests package -B -q -T 1C && \
    mv /build/target/orders-0.0.1-SNAPSHOT.jar /app.jar

# Package Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    shadow-utils \
    findutils \
    && \
    dnf clean all

# Use curl-full to support "telnet://" scheme
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

COPY --chown=appuser:appuser src/orders/ATTRIBUTION.md ./LICENSES.md
COPY --chown=appuser:appuser --from=build-env /app.jar .

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
