# syntax=docker/dockerfile:1

# --- Build stage : compile + package fast-jar ---
FROM maven:3.9.16-eclipse-temurin-21 AS build
WORKDIR /build
# Cache des dependances : copie du pom seul d'abord
COPY pom.xml .
RUN mvn -B -ntp dependency:go-offline
COPY src ./src
RUN mvn -B -ntp clean package -DskipTests

# --- Runtime stage : JRE leger optimise Quarkus ---
FROM registry.access.redhat.com/ubi9/openjdk-21-runtime:latest
ENV LANGUAGE='en_US:en'
COPY --from=build --chown=185 /build/target/quarkus-app/lib/ /deployments/lib/
COPY --from=build --chown=185 /build/target/quarkus-app/*.jar /deployments/
COPY --from=build --chown=185 /build/target/quarkus-app/app/ /deployments/app/
COPY --from=build --chown=185 /build/target/quarkus-app/quarkus/ /deployments/quarkus/
EXPOSE 8082
USER 185
ENV JAVA_OPTS_APPEND="-Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager"
ENV JAVA_APP_JAR="/deployments/quarkus-run.jar"
