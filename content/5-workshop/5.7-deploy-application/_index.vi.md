---
title: "5.7. Triển khai ứng dụng"
weight: 7
chapter: false
---

# 5.5. Triển khai ECS Fargate và ALB

## 5.5.1. Target Group và Application Load Balancer

Tạo **EC2 → Target Groups → Create target group**:

- Target type: **IP addresses**.
- Name: `enggo-backend-tg`.
- Protocol/Port: HTTP/8080.
- VPC: `enggo-vpc`.
- Health check: `/api/v1/health` hoặc `/actuator/health`.

Trong Attributes, bật sticky session duration-based 86400 giây nếu PvP yêu cầu giữ phiên.

Tạo **Load Balancers → Application Load Balancer**:

- Name: `enggo-alb`.
- Scheme: Internet-facing.
- VPC: `enggo-vpc`.
- Subnets: `enggo-public-subnet-1a`, `enggo-public-subnet-1c`.
- Security group: `enggo-alb-sg`.
- Listener HTTP/80 forward tới `enggo-backend-tg`.

## 5.5.2. Task Definition

Tạo `task-definition.json`:

```json
{
  "family": "enggo-backend-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::YOUR_ACCOUNT_ID:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::YOUR_ACCOUNT_ID:role/enggoTaskRole",
  "containerDefinitions": [{
    "name": "enggo-backend",
    "image": "YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest",
    "portMappings": [{"containerPort": 8080, "hostPort": 8080, "protocol": "tcp"}],
    "environment": [
      {"name": "SPRING_PROFILES_ACTIVE", "value": "prod"},
      {"name": "DB_HOST", "value": "enggo-db.xxxx.ap-northeast-1.rds.amazonaws.com"},
      {"name": "SPRING_REDIS_HOST", "value": "enggo-redis.xxxx.ng.0001.apne1.cache.amazonaws.com"},
      {"name": "SPRING_REDIS_PORT", "value": "6379"}
    ],
    "secrets": [
      {"name": "DB_PASSWORD", "valueFrom": "arn:aws:ssm:ap-northeast-1:YOUR_ACCOUNT_ID:parameter/config/enggo-backend/DB_PASSWORD"},
      {"name": "JWT_SIGNER_KEY", "valueFrom": "arn:aws:ssm:ap-northeast-1:YOUR_ACCOUNT_ID:parameter/config/enggo-backend/JWT_SIGNER_KEY"},
      {"name": "AWS_S3_ACCESS_KEY", "valueFrom": "arn:aws:ssm:ap-northeast-1:YOUR_ACCOUNT_ID:parameter/config/enggo-backend/AWS_S3_ACCESS_KEY"},
      {"name": "AWS_S3_SECRET_KEY", "valueFrom": "arn:aws:ssm:ap-northeast-1:YOUR_ACCOUNT_ID:parameter/config/enggo-backend/AWS_S3_SECRET_KEY"}
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/enggo-backend",
        "awslogs-region": "ap-northeast-1",
        "awslogs-stream-prefix": "ecs"
      }
    }
  }]
}
```

```bash
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

## 5.5.3. ECS Cluster và Service

```bash
aws ecs create-cluster --cluster-name enggo-cluster --region ap-northeast-1
aws ecs create-service \
  --cluster enggo-cluster \
  --service-name enggo-backend-service \
  --task-definition enggo-backend-task \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[$SUBNET_PRI_1A,$SUBNET_PRI_1C],securityGroups=[$BACKEND_SG_ID],assignPublicIp=DISABLED}" \
  --load-balancers "targetGroupArn=$TARGET_GROUP_ARN,containerName=enggo-backend,containerPort=8080" \
  --region ap-northeast-1
```

**Nghiệm thu:** hai task chuyển `Healthy`, `GET /api/v1/topics` trả HTTP 200 và WebSocket `/ws/pvp` trả handshake 101.
