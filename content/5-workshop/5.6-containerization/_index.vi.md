---
title: "5.6. Đóng gói ứng dụng"
weight: 6
chapter: false
---

# 5.4. Xây dựng và triển khai ứng dụng

## 5.4.1. Kiến trúc Backend

`enggo-backend` sử dụng Spring Boot 3.4.3 và Java 21:

```text
config/       JWT, CORS, AWS S3, WebSocket STOMP, Redis
controller/   /api/v1/auth, /api/v1/topics, /api/v1/vocabularies, /ws/pvp
service/      nghiệp vụ, ELO PvP, spaced repetition, upload media
repository/   Spring Data JPA
entity/       ánh xạ bảng tienganh_app
```

## 5.4.2. `application-prod.yml`

```yaml
server:
  port: 8080
  servlet:
    context-path: /
spring:
  application:
    name: enggo-backend
  profiles:
    active: prod
  datasource:
    url: jdbc:mysql://${DB_HOST:localhost}:3306/tienganh_app?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
    username: admin
    password: ${DB_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver
  data:
    redis:
      host: ${SPRING_REDIS_HOST:localhost}
      port: ${SPRING_REDIS_PORT:6379}
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: false
aws:
  region: ${AWS_REGION:ap-northeast-1}
  s3:
    bucket-name: ${AWS_S3_BUCKET:enggo-media-bucket}
    access-key: ${AWS_S3_ACCESS_KEY}
    secret-key: ${AWS_S3_SECRET_KEY}
jwt:
  signerKey: ${JWT_SIGNER_KEY}
```

## 5.4.3. Dockerfile multi-stage

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

## 5.4.4. Build và push lên ECR

```bash
aws ecr create-repository --repository-name enggo-backend --region ap-northeast-1
aws ecr get-login-password --region ap-northeast-1 |
  docker login --username AWS --password-stdin \
  YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com
docker build -t enggo-backend:latest .
docker tag enggo-backend:latest \
  YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest
docker push YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest
```

**Nghiệm thu:** image `enggo-backend:latest` xuất hiện trong ECR và sẵn sàng cho ECS.
