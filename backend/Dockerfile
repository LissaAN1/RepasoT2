# =============================================================================
# Taller Evaluativo 2 - Fase 2: Empaquetamiento Inmutable Multi-Stage
# Backend: studytrack-api
#
# Complete los TODOs siguiendo las buenas prácticas vistas en el curso:
# reducción de peso, cache de dependencias, usuario no root y ausencia
# total del SDK/compilador en la imagen final de runtime.
# =============================================================================

# ---------- Stage 1: Builder ----------
# TODO
# Nota: el tag "maven:3.9.6-alpine" no existe en Docker Hub; el equivalente
# Alpine con Maven 3.9.6 + JDK 17 es el siguiente (tag fijo, nunca :latest).
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder
WORKDIR /app

# Cache de dependencias: se copia primero solo el pom.xml
COPY pom.xml .
RUN mvn -B -q dependency:go-offline

COPY src ./src
RUN mvn -B package -DskipTests

# ---------- Stage 2: Runtime ----------
# TODO
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Usuario no root
RUN addgroup -S appuser && adduser -S appuser -G appuser

# Solo se copia el .jar (sin SDK, sin Maven, sin código fuente)
COPY --from=builder --chown=appuser:appuser /app/target/studytrack-api.jar app.jar

USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
