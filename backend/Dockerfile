# ---------------------------------------------------------------------------
# Stage 1: Build
# maven:3.9.9-eclipse-temurin-22 bundles Maven with the exact JDK the project
# targets (Java 22), so the compiled bytecode matches the runtime JRE below.
# This stage is discarded from the final image, so its size doesn't matter.
# ---------------------------------------------------------------------------
FROM maven:3.9.9-eclipse-temurin-22 AS build
WORKDIR /app

# Copy only the POM first so dependency resolution is cached in its own layer
# and isn't re-run every time application source code changes.
COPY pom.xml .
RUN mvn -B dependency:go-offline

COPY src ./src
RUN mvn -B clean package -DskipTests

# ---------------------------------------------------------------------------
# Stage 2: Runtime
# eclipse-temurin:22-jre-alpine ships only the JRE (no compiler/build tools)
# on top of Alpine Linux, giving a small image and a minimal attack surface
# for what is purely a "run this jar" container.
# ---------------------------------------------------------------------------
FROM eclipse-temurin:22-jre-alpine AS runtime
WORKDIR /app

# Run as a non-root user rather than the image default root.
RUN addgroup -S spring && adduser -S spring -G spring
COPY --from=build /app/target/*.jar app.jar
RUN chown spring:spring app.jar
USER spring

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
    CMD wget -qO- http://localhost:8080/v3/api-docs || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]