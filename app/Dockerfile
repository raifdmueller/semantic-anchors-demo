# syntax=docker/dockerfile:1
#
# Multi-stage build: compiles the application with a full JDK, then runs
# it with a minimal JRE-only base image.
#
# Build (from the app/ directory):
#   docker build -t quarkus/semantic-anchors-demo .
#
# Run:
#   docker run -i --rm -p 8080:8080 quarkus/semantic-anchors-demo
#
# For hardened production deployments, add these flags to `docker run`:
#   --read-only --tmpfs /tmp --security-opt no-new-privileges:true --cap-drop ALL

###############################################################################
# Stage 1 – Build (Maven + JDK 25)
###############################################################################
FROM maven:3.9.15-eclipse-temurin-25 AS builder

WORKDIR /build

# Copy POM first so dependency downloads are cached in a separate layer
# and only re-run when pom.xml changes.
COPY pom.xml ./
RUN mvn dependency:go-offline -B -q

# Copy source code and produce the Quarkus fast-jar
COPY src src/
RUN mvn package -DskipTests -B -q

###############################################################################
# Stage 2 – Runtime (JRE only, no build tools, no JDK)
###############################################################################
FROM registry.access.redhat.com/ubi9/openjdk-25-runtime:1.24

ENV LANGUAGE='en_US:en'

# Copy only the built artefacts; nothing from the build stage leaks into
# this image (no JDK, no Maven, no source code, no test dependencies).
COPY --chown=185:185 --from=builder /build/target/quarkus-app/lib/     /deployments/lib/
COPY --chown=185:185 --from=builder /build/target/quarkus-app/*.jar    /deployments/
COPY --chown=185:185 --from=builder /build/target/quarkus-app/app/     /deployments/app/
COPY --chown=185:185 --from=builder /build/target/quarkus-app/quarkus/ /deployments/quarkus/

EXPOSE 8080

# Run as the non-root jboss user (uid 185) provided by the base image.
USER 185

ENV JAVA_OPTS_APPEND="-Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager"
ENV JAVA_APP_JAR="/deployments/quarkus-run.jar"

ENTRYPOINT [ "/opt/jboss/container/java/run/run-java.sh" ]
