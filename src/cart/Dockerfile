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

COPY src/cart/.mvn .mvn
COPY src/cart/mvnw .
COPY src/cart/pom.xml .

RUN ./mvnw dependency:go-offline -B -q

COPY src/cart/src ./src

RUN ./mvnw -DskipTests package -B -q -T 1C && \
    mv /build/target/carts-0.0.1-SNAPSHOT.jar /app.jar

# Package Stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    findutils \
    shadow-utils \
    && \
    dnf clean all

RUN groupadd -r appuser && useradd -r -g appuser -d /app -m appuser

ENV JAVA_TOOL_OPTIONS=
ENV SPRING_PROFILES_ACTIVE=prod

WORKDIR /app
COPY --chown=appuser:appuser src/cart/ATTRIBUTION.md ./LICENSES.md
COPY --chown=appuser:appuser --from=build-env /app.jar .

USER appuser

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
