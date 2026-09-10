---
title: "Docker Multi-stage Build"
weight: 5
chapter: false
pre: "<b>5.4.5. </b>"
date: 2026-09-10
---

# Docker Multi-stage Build

Tạo `Dockerfile` tại thư mục gốc repository:

```dockerfile
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN chmod +x ./mvnw
RUN ./mvnw clean package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```
