# Build stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build
RUN dnf install -y java-21-amazon-corretto-devel maven findutils && \
    dnf clean all
ENV JAVA_HOME=/usr/lib/jvm/java-21-amazon-corretto
ENV PATH="${JAVA_HOME}/bin:${PATH}"
ARG MAVEN_OPTS
WORKDIR /build
COPY src/orders/pom.xml ./pom.xml
COPY src/orders/src ./src
RUN mvn --version && \
    mvn -Dmaven.resolver.transport=wagon package -DskipTests -B -T 1C && \
    cp target/orders-0.0.1-SNAPSHOT.jar /build/app.jar

# Runtime stage
FROM public.ecr.aws/amazonlinux/amazonlinux:2023
RUN dnf install -y java-21-amazon-corretto-headless shadow-utils findutils && \
    dnf clean all && \
    groupadd -r appuser && useradd -r -g appuser -d /app appuser
WORKDIR /app
COPY --chown=appuser:appuser --from=build /build/app.jar /app/app.jar
USER appuser
EXPOSE 8080
ENTRYPOINT ["sh", "-c", "exec java $JAVA_OPTS -jar /app/app.jar"]
