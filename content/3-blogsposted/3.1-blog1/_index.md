---
title: "3.1. Blog 1 - Spring Boot Deployment on ECS Fargate"
date: 2026-09-10
weight: 1
chapter: false
---

# SPRING BOOT BACKEND DEPLOYMENT ON AWS ECS FARGATE WITH LOAD BALANCING

This blog introduces the deployment of a Spring Boot backend application on AWS ECS Fargate with load balancer configuration, database management, and automated CI/CD pipeline setup.

---

## 1. Architecture Overview

The Spring Boot Backend on AWS ECS Fargate solution provides a complete cloud-native architecture, enabling backend applications to scale automatically without server management:

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

## 2. Key AWS Services

### Amazon ECS Fargate
- **Serverless container orchestration** - no EC2 instance management
- **Auto-scaling** - automatically scales tasks based on CPU/Memory metrics
- **CloudWatch integration** - automatic logging and monitoring

### Application Load Balancer (ALB)
- **Health checks** - monitors task health status
- **Request routing** - distributes traffic evenly to tasks
- **HTTPS/TLS** - secure communication with ACM certificates

### Amazon RDS
- **Managed database** - PostgreSQL/MySQL managed by AWS
- **Multi-AZ deployment** - high availability and disaster recovery
- **Automated backups** - automatic backups and point-in-time recovery

### AWS CodeBuild & CodePipeline
- **Automated build** - build Docker image when code is pushed
- **Automated deployment** - deploy to ECS Fargate automatically
- **CI/CD pipeline** - fully automated workflow

---

## 3. Deployment Process

### Step 1: Infrastructure Preparation
```bash
# Create VPC, Subnets, Security Groups
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create RDS Database
aws rds create-db-instance --db-instance-identifier enggo-db \
  --db-instance-class db.t3.small --engine mysql

# Configure IAM roles for ECS
aws iam create-role --role-name ecsTaskExecutionRole
```

### Step 2: Build and Push Docker Image
```bash
# Build Docker Image
docker build -t enggo-backend:latest .

# Tag for ECR
docker tag enggo-backend:latest <account-id>.dkr.ecr.<region>.amazonaws.com/enggo-backend:latest

# Push to ECR
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/enggo-backend:latest
```

### Step 3: Create ECS Task Definition
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

### Step 4: Deploy ECS Service
```bash
# Create ECS Service
aws ecs create-service \
  --cluster enggo-cluster \
  --service-name enggo-service \
  --task-definition enggo-backend-task \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp=ENABLED}"
```

### Step 5: Configure Load Balancer
```bash
# Create Target Group
aws elbv2 create-target-group \
  --name enggo-tg \
  --protocol HTTP \
  --port 8080 \
  --vpc-id vpc-xxx \
  --target-type ip

# Configure health check
aws elbv2 modify-target-group \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --health-check-path /actuator/health \
  --health-check-interval-seconds 30
```

---

## 4. Best Practices

### Security
- ✅ Use Security Groups to restrict traffic
- ✅ Encrypt RDS database connections
- ✅ Use IAM roles with least privilege
- ✅ Enable CloudTrail logging for audit

### Performance
- ✅ Optimize Spring Boot startup time
- ✅ Use connection pooling for databases
- ✅ Enable caching with ElastiCache (optional)
- ✅ Monitor CPU/Memory metrics

### Cost Optimization
- ✅ Use Fargate Spot instances for non-critical workloads
- ✅ Optimize resource allocation (CPU/Memory)
- ✅ Implement auto-scaling policies
- ✅ Monitor CloudWatch costs

---

## 5. Results and Benefits

### Technical
- ✅ Fully cloud-native backend application
- ✅ High availability with multi-AZ deployment
- ✅ Auto-scaling based on metrics
- ✅ Automated CI/CD pipeline
- ✅ Zero-downtime deployments

### Business
- ✅ Reduced operational overhead (no server management)
- ✅ Pay-per-use pricing model
- ✅ Easy to scale up/down
- ✅ Fast time-to-market for new features

---

## 6. Additional References

- [AWS ECS Fargate Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/)
- [Spring Boot on AWS Best Practices](https://aws.amazon.com/blogs/mobile/spring-boot-on-aws/)
- [Application Load Balancer Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [AWS CodeBuild for Docker](https://docs.aws.amazon.com/codebuild/latest/userguide/)

---

**Published:** April 2026  
**Last Updated:** May 2026
