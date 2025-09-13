# Multi-stage build for optimized image size
FROM maven:3.8.6-eclipse-temurin-17 AS build

# Set working directory
WORKDIR /app

# Copy pom.xml and download dependencies (for better caching)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code
COPY src ./src

# Build the application (skip tests for faster build)
RUN mvn clean package -DskipTests

# Runtime stage with minimal JRE image
FROM eclipse-temurin:17-jre-alpine

# Create non-root user for security
RUN addgroup --system javauser && adduser -S -s /usr/sbin/nologin -G javauser javauser

# Set working directory
WORKDIR /opt/app

# Copy JAR file from build stage
COPY --from=build /app/target/chat-app-*.jar app.jar

# Change ownership to non-root user
RUN chown -R javauser:javauser .

# Switch to non-root user
USER javauser

# Expose port (Render uses PORT environment variable)
EXPOSE ${PORT:-8080}

# Health check for better monitoring
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:${PORT:-8080}/actuator/health || exit 1

# Start the application with optimized JVM settings
ENTRYPOINT ["java", \
    "-Djava.security.egd=file:/dev/./urandom", \
    "-Dserver.port=${PORT:-8080}", \
    "-Xms256m", \
    "-Xmx512m", \
    "-jar", \
    "app.jar"]
