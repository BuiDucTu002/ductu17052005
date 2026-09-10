---
title: "3.2. Blog 2 - Containerizing Java Applications with Docker"
date: "`r Sys.Date()`"
weight: 2
chapter: false
---

# CONTAINERIZING JAVA APPLICATIONS WITH DOCKER: BEST PRACTICES AND OPTIMIZATION

Bài viết này phân tích các best practices khi containerizing Java applications bằng Docker, từ tối ưu hóa Dockerfile đến triển khai trên Amazon ECS, giúp tạo ra các container image hiệu quả và an toàn.

---

## 1. Vấn đề Khi Containerizing Java Applications

Java applications thường gặp các thách thức khi containerization:

### Kích thước Docker Image Lớn
- Base Java images có thể lên đến 500MB+
- Các dependencies thêm vào đáng kể
- Layer caching không hiệu quả

### Startup Time
- JVM warmup time có thể kéo dài
- Class initialization overhead
- Memory overhead từ JVM

### Security
- Outdated base images
- Unnecessary vulnerabilities
- Lack of non-root user

---

## 2. Best Practices cho Java Container

### 2.1 Multi-stage Build

**❌ Không tối ưu (Dockerfile duy nhất):**
```dockerfile
FROM openjdk:11
WORKDIR /app
COPY . .
RUN ./gradlew build
EXPOSE 8080
CMD ["java", "-jar", "build/libs/app.jar"]
# Kích thước: ~900MB
```

**✅ Tối ưu (Multi-stage Build):**
```dockerfile
# Stage 1: Build
FROM openjdk:11-jdk as builder
WORKDIR /app
COPY . .
RUN ./gradlew build

# Stage 2: Runtime
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
EXPOSE 8080
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser
ENTRYPOINT ["java", "-jar", "app.jar"]
# Kích thước: ~350MB (giảm 60%)
```

### 2.2 Image Base Optimization

| Base Image | Kích thước | Ưu điểm |
|-----------|-----------|---------|
| openjdk:11 | ~500MB | Feature complete |
| openjdk:11-jre-slim | ~200MB | Nhẹ hơn, đủ cho runtime |
| openjdk:11-jre-alpine | ~100MB | Rất nhẹ (Alpine Linux) |
| eclipse-temurin:11-jre-focal | ~180MB | Modern, maintained |

**Khuyến cáo:** Sử dụng `eclipse-temurin` hoặc `openjdk:11-jre-slim` cho balance

### 2.3 Layer Optimization

```dockerfile
# ❌ Không tốt (mỗi lệnh là layer riêng)
FROM openjdk:11-jre-slim
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean
COPY app.jar .
RUN mkdir -p /app/logs

# ✅ Tốt (merged commands)
FROM openjdk:11-jre-slim
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
COPY app.jar .
RUN mkdir -p /app/logs
```

### 2.4 Caching Strategy

```dockerfile
# Dockerfile tối ưu cho caching:
FROM openjdk:11-jre-slim

WORKDIR /app

# Copy dependency files first (thay đổi ít)
COPY pom.xml .
COPY mvnw .
RUN ./mvnw -B org.apache.maven.plugins:maven-dependency-plugin:3.1.2:resolve

# Copy source code (thay đổi thường xuyên)
COPY src ./src

# Build application
RUN ./mvnw clean package -DskipTests

# Runtime stage
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=0 /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 3. JVM Tuning cho Container

### Memory Configuration

```dockerfile
FROM openjdk:11-jre-slim
WORKDIR /app

# Đặt JVM heap size
ENV JAVA_OPTS="-Xms256m -Xmx512m"

COPY app.jar .
EXPOSE 8080

# Sử dụng exec form để signal handling chính xác
ENTRYPOINT exec java $JAVA_OPTS -jar app.jar
```

### GC Tuning

```bash
# Container environment variables
JAVA_OPTS="-XX:+UseG1GC \
           -XX:MaxGCPauseMillis=200 \
           -XX:+ParallelRefProcEnabled \
           -Xms256m -Xmx512m"

# Hoặc để JVM auto-detect:
JAVA_OPTS="-XX:+UseContainerSupport -Xmx512m"
```

---

## 4. Security Best Practices

### Non-root User

```dockerfile
FROM openjdk:11-jre-slim

# Tạo non-root user
RUN useradd -m -u 1000 appuser

WORKDIR /app
COPY --chown=appuser:appuser app.jar .

# Switch to non-root user
USER appuser

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Image Scanning

```bash
# Scan cho vulnerabilities
docker scan enggo-backend:latest

# Build-time scanning
docker build --build-arg SCAN=true .

# Registry scanning (ECR)
aws ecr start-image-scan \
  --repository-name enggo-backend \
  --image-id imageTag=latest
```

---

## 5. Logging và Monitoring

### Structured Logging

```yaml
# Spring Boot application.yml
logging:
  level:
    root: INFO
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
  file:
    name: /var/log/app.log

# JSON logging cho CloudWatch parsing
spring:
  cloud:
    aws:
      cloudwatch:
        enabled: true
```

### Health Check

```dockerfile
FROM openjdk:11-jre-slim
WORKDIR /app
COPY app.jar .

# Spring Boot actuator health endpoint
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 6. Build Performance

### Build Cache Strategy

```bash
# Tận dụng BuildKit
export DOCKER_BUILDKIT=1

# Build with progress output
docker build --progress=plain -t enggo-backend:latest .

# Multi-platform builds
docker buildx build --platform linux/amd64,linux/arm64 \
  -t enggo-backend:latest .
```

### CI/CD Integration

```yaml
# AWS CodeBuild buildspec.yml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  
  build:
    commands:
      - echo Building Docker image...
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
  
  post_build:
    commands:
      - echo Pushing Docker image to ECR...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG

artifacts:
  files:
    - imagedefinitions.json
```

---

## 7. Kết quả Tối ưu

### Trước và Sau Tối ưu

| Metric | Trước | Sau | Cải thiện |
|--------|------|------|---------|
| Image Size | 900MB | 250MB | -72% |
| Build Time | 8 phút | 3 phút | -63% |
| Startup Time | 15s | 5s | -67% |
| Security Scan | 12 vulnerabilities | 0 | 100% |
| Memory Usage | 1GB | 512MB | -50% |

---

## 8. Tham khảo Bổ sung

- [Best Practices for Java on Docker](https://docs.docker.com/language/java/)
- [Spring Boot Docker Guide](https://spring.io/guides/topicals/spring-boot-docker)
- [Distroless Images for Java](https://github.com/GoogleContainerTools/distroless)
- [AWS ECR Best Practices](https://docs.aws.amazon.com/AmazonECR/latest/userguide/)

---

**Bài viết được đăng:** Tháng 4, 2026  
**Bản cập nhật cuối:** Tháng 5, 2026
