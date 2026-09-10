---
title: "5.6.1. Build Docker Image"
weight: 1
chapter: false
---

# 5.6.1. Build Docker Image

```dockerfile
FROM maven:3.9.6-eclipse-temurin-21-alpine AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build và chạy thử:

```bash
docker build -t enggo-backend:latest .
docker run --rm -p 8080:8080 enggo-backend:latest
```

Kiểm tra health endpoint trước khi push image.
