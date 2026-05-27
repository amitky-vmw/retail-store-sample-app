# Build Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

RUN dnf --setopt=install_weak_deps=False install -q -y \
    maven \
    java-21-amazon-corretto-headless \
    findutils \
    tar \
    gzip \
    && \
    dnf clean all

WORKDIR /build

COPY src/orders/.mvn .mvn
COPY src/orders/mvnw .
COPY src/orders/pom.xml .

RUN ./mvnw dependency:go-offline -B -q

COPY src/orders/src ./src

RUN ./mvnw -DskipTests package -B -q && \
    mv /build/target/orders-0.0.1-SNAPSHOT.jar /app.jar

# Runtime Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    shadow-utils \
    findutils \
    && \
    dnf clean all

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
