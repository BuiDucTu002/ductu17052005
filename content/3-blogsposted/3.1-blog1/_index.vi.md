---
title: "3.1. Blog 1 - Spring Boot Deployment on ECS Fargate"
date: "`r Sys.Date()`"
weight: 1
chapter: false
---

# SPRING BOOT BACKEND DEPLOYMENT ON AWS ECS FARGATE WITH LOAD BALANCING

Bài viết này giới thiệu về việc triển khai một ứng dụng Spring Boot backend trên nền tảng AWS ECS Fargate với cấu hình load balancer, quản lý database, và thiết lập CI/CD pipeline tự động.

---

## 1. Tóm tắt Kiến trúc

Giải pháp Spring Boot Backend on AWS ECS Fargate cung cấp một kiến trúc cloud-native hoàn chỉnh, cho phép các ứng dụng backend có khả năng mở rộng tự động mà không cần quản lý máy chủ:

```
┌─────────────────────────────────────────────────────┐
│           Client Applications                       │
└────────────────────┬────────────────────────────────┘
                     ↓
         Amazon Route 53 (DNS)
                     ↓
    AWS Certificate Manager (ACM - HTTPS)
                     ↓
    Application Load Balancer (ALB)
                     ↓
┌─────────────────────────────────────────────────────┐
│   Amazon ECS Fargate Cluster                        │
│  ┌──────────────────────────────────────────────┐  │
│  │  Spring Boot Container Tasks (Auto-scaling)  │  │
│  └──────────────────────────────────────────────┘  │
└─────┬──────────────────────────────────────────┬───┘
      ↓                                           ↓
Amazon RDS (Database)              Amazon S3 (Storage)
      ↓
AWS CodeBuild & CodePipeline (CI/CD)
      ↓
GitHub Repository
```

---

## 2. Các Dịch vụ AWS Chính

### Amazon ECS Fargate
- **Serverless container orchestration** - không cần quản lý EC2 instances
- **Auto-scaling** - tự động scale tasks dựa trên CPU/Memory metrics
- **CloudWatch integration** - logging và monitoring tự động

### Application Load Balancer (ALB)
- **Health check** - kiểm tra tình trạng các tasks
- **Request routing** - phân phối traffic đều tới các tasks
- **HTTPS/TLS** - bảo mật giao tiếp với ACM certificates

### Amazon RDS
- **Managed database** - PostgreSQL/MySQL được quản lý bởi AWS
- **Multi-AZ deployment** - high availability và disaster recovery
- **Automated backups** - sao lưu tự động và point-in-time recovery

### AWS CodeBuild & CodePipeline
- **Automated build** - build Docker image khi code được push
- **Automated deployment** - deploy to ECS Fargate tự động
- **CI/CD pipeline** - workflow hoàn toàn tự động

---

## 3. Quá trình Triển khai

### Bước 1: Chuẩn bị Cơ sở hạ tầng
```bash
# Tạo VPC, Subnets, Security Groups
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Tạo RDS Database
aws rds create-db-instance --db-instance-identifier enggo-db \
  --db-instance-class db.t3.small --engine mysql

# Cấu hình IAM roles cho ECS
aws iam create-role --role-name ecsTaskExecutionRole
```

### Bước 2: Build và Push Docker Image
```bash
# Build Docker Image
docker build -t enggo-backend:latest .

# Tag cho ECR
docker tag enggo-backend:latest <account-id>.dkr.ecr.<region>.amazonaws.com/enggo-backend:latest

# Push to ECR
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/enggo-backend:latest
```

### Bước 3: Tạo ECS Task Definition
```json
{
  "family": "enggo-backend-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [{
    "name": "enggo-backend",
    "image": "<account-id>.dkr.ecr.<region>.amazonaws.com/enggo-backend:latest",
    "portMappings": [{
      "containerPort": 8080,
      "hostPort": 8080,
      "protocol": "tcp"
    }],
    "environment": [
      {
        "name": "DB_HOST",
        "value": "<rds-endpoint>"
      }
    ]
  }]
}
```

### Bước 4: Triển khai ECS Service
```bash
# Tạo ECS Service
aws ecs create-service \
  --cluster enggo-cluster \
  --service-name enggo-service \
  --task-definition enggo-backend-task \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp=ENABLED}"
```

### Bước 5: Cấu hình Load Balancer
```bash
# Tạo Target Group
aws elbv2 create-target-group \
  --name enggo-tg \
  --protocol HTTP \
  --port 8080 \
  --vpc-id vpc-xxx \
  --target-type ip

# Cấu hình health check
aws elbv2 modify-target-group \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --health-check-path /actuator/health \
  --health-check-interval-seconds 30
```

---

## 4. Best Practices

### Security
- ✅ Sử dụng Security Groups để restrict traffic
- ✅ Mã hóa RDS database connection
- ✅ Sử dụng IAM roles với least privilege
- ✅ Enable CloudTrail logging cho audit

### Performance
- ✅ Tối ưu hóa Spring Boot startup time
- ✅ Sử dụng connection pooling cho database
- ✅ Enable caching với ElastiCache (optional)
- ✅ Monitor CPU/Memory metrics

### Cost Optimization
- ✅ Sử dụng Fargate Spot instances cho non-critical workloads
- ✅ Tối ưu hóa resource allocation (CPU/Memory)
- ✅ Implement auto-scaling policies
- ✅ Monitor CloudWatch costs

---

## 5. Kết quả và Lợi ích

### Kỹ thuật
- ✅ Ứng dụng backend hoàn toàn cloud-native
- ✅ High availability với multi-AZ deployment
- ✅ Auto-scaling dựa trên metrics
- ✅ Tự động CI/CD pipeline
- ✅ Zero-downtime deployments

### Kinh doanh
- ✅ Giảm operational overhead (không quản lý servers)
- ✅ Pay-per-use pricing model
- ✅ Dễ dàng scale lên/xuống
- ✅ Nhanh time-to-market cho features mới

---

## 6. Tham khảo Bổ sung

- [AWS ECS Fargate Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/)
- [Spring Boot on AWS Best Practices](https://aws.amazon.com/blogs/mobile/spring-boot-on-aws/)
- [Application Load Balancer Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [AWS CodeBuild for Docker](https://docs.aws.amazon.com/codebuild/latest/userguide/)

---

**Bài viết được đăng:** Tháng 4, 2026  
**Bản cập nhật cuối:** Tháng 5, 2026
