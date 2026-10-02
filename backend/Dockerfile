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
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder

WORKDIR /app


COPY pom.xml .

RUN mvn -B -q dependency:go-offline

COPY src ./src

RUN mvn -B package -DskipTests


# ---------- Stage 2: Runtime ----------
# TODO
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app


RUN addgroup -S appuser && adduser -S appuser -G appuser


COPY --from=builder --chown=appuser:appuser /app/target/studytrack-api.jar app.jar

USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]