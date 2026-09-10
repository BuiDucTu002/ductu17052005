---
title: "2. Proposal"
date: "`r Sys.Date()`"
weight: 2
chapter: false
---

# Proposal

# Enggo-Backend Cloud-Native System on AWS

---

# 1. Summary

Enggo-Backend is a modern backend application platform built with Spring Boot and designed for deployment on AWS managed services. The system provides complete RESTful APIs, user authentication, data management, file handling, and advanced features through a scalable cloud platform.

The application is developed using **Java Spring Boot**, **Spring Data JPA**, and **Spring Security**, with **PostgreSQL** or **MySQL** as the database. The application is containerized with **Docker** and deployed on **Amazon ECS Fargate** behind an **Application Load Balancer (ALB)**. Docker Images are stored in **Amazon ECR**, data files are stored in **Amazon S3**, and **AWS CodeBuild** automatically builds and deploys the latest version whenever code is pushed to GitHub.

The deployment environment also uses **Amazon RDS** for database management, **Amazon Route 53** for domain management, **AWS Certificate Manager (ACM)** for HTTPS encryption, **Amazon CloudWatch** for monitoring, **AWS IAM** for access control, and **Amazon VPC** for network security. This architecture provides automated deployment, centralized storage, simplified management, and scalable cloud infrastructure suitable for small to medium-sized backend applications.

---

# 2. Problem Statement

## Current Issues

Many traditional backend applications face the following challenges:
- Complex local database management that is difficult to scale
- Manual deployment requiring significant operational effort
- Difficulties managing resources and infrastructure costs
- High downtime during updates or bug fixes
- Difficult to maintain and expand as user volume increases

## Solution

The proposed solution is to develop a cloud-native backend platform using AWS managed services.

The Enggo-Backend system provides:
- **Complete RESTful APIs** for frontend applications
- **Secure authentication** using Spring Security and JWT
- **Centralized data management** via Amazon RDS
- **Reliable file storage** via Amazon S3
- **Automatic deployment** through CI/CD pipeline
- **Auto-scaling** capability with Amazon ECS Auto Scaling
- **Real-time monitoring** via Amazon CloudWatch
- **Optimized security** through AWS IAM and VPC

## Benefits

The proposed architecture delivers the following benefits:

- ✅ Simplified backend application deployment
- ✅ Fully automated CI/CD workflow
- ✅ Auto-scaling cloud infrastructure
- ✅ Secure HTTPS/TLS communication
- ✅ Reliable data storage with automatic backups
- ✅ Simplified application maintenance
- ✅ Significantly reduced operational effort
- ✅ Easy scalability for future growth

---

# 3. Solution Architecture

## Architecture Diagram

The application uses a cloud-native architecture deployed on AWS managed services:

```
Client Applications
        ↓
Amazon Route 53 (DNS)
        ↓
AWS Certificate Manager (ACM - HTTPS)
        ↓
Application Load Balancer (ALB)
        ↓
Amazon ECS Fargate (Container Orchestration)
        ↓
┌───────────────────────────────────┐
│   Spring Boot Application          │
│   (Java Backend Service)           │
└───────────────────────────────────┘
        ↓         ↓         ↓
   Amazon    Amazon S3    Amazon
   RDS      (Storage)     ECR
   (DB)    (Image Files) (Registry)
   ↓
AWS CodeBuild & CodePipeline (CI/CD)
        ↓
GitHub Repository
```

## AWS Services Used

| Service | Function |
|---------|----------|
| **Amazon VPC** | Private secure virtual network |
| **AWS IAM** | Access control management |
| **Amazon ECS Fargate** | Serverless container management |
| **Amazon ECR** | Docker Image repository |
| **Amazon RDS** | Managed database service |
| **Amazon S3** | Remote data storage |
| **Application Load Balancer (ALB)** | Load balancing |
| **Amazon Route 53** | DNS domain management |
| **AWS Certificate Manager (ACM)** | HTTPS/TLS certificates |
| **AWS CodeBuild** | Automatic Docker Image building |
| **AWS CodePipeline** | Automated CI/CD workflow |
| **Amazon CloudWatch** | Monitoring and logging |

## Component Design

### Backend

- **Language:** Java
- **Framework:** Spring Boot, Spring Data JPA, Spring Security
- **API:** RESTful API, JSON
- **Authentication:** JWT Token, Spring Security

### Database

- **Amazon RDS:** PostgreSQL or MySQL
- **Automatic Backup:** Automated Backup
- **High Availability:** Multi-AZ deployment

### File Storage

- **Amazon S3:** File data storage
- **CloudFront:** CDN for speed optimization (optional)

### Container Platform

- **Docker:** Containerization
- **Amazon ECS Fargate:** Orchestration
- **Amazon ECR:** Docker Registry

### Deployment Pipeline

```
GitHub Repository (Push code)
            ↓
AWS CodeBuild (Build Docker Image)
            ↓
Amazon ECR (Store Docker Image)
            ↓
AWS CodeDeploy/ECS (Deploy to Fargate)
            ↓
Application Live
```

---

# 4. Technical Implementation

## Implementation Stages

The project is implemented through the following stages:

1. **Week 1:** Research AWS architecture, analyze Enggo-Backend project
2. **Week 2:** Design architecture, configure VPC, prepare Amazon RDS
3. **Week 3:** Backend development, configure Docker, build Docker Image
4. **Week 4:** Deploy Amazon ECS Fargate, ALB, Route 53, ACM
5. **Week 5:** Configure CI/CD, monitor with CloudWatch, system testing

## Technical Requirements

### Language & Framework

- Java
- Spring Boot
- Spring Data JPA
- Spring Security

### Database

- PostgreSQL / MySQL
- Amazon RDS

### AWS Services

- Amazon VPC, AWS IAM
- Amazon ECS Fargate, Amazon ECR
- Amazon RDS, Amazon S3
- Application Load Balancer
- Amazon Route 53, AWS Certificate Manager
- AWS CodeBuild, AWS CodePipeline
- Amazon CloudWatch

### Development Tools

- IntelliJ IDEA / VS Code
- Git & GitHub
- Docker Desktop
- Maven / Gradle
- AWS CLI

---

# 5. Roadmap & Milestones

### Week 1 – Research & Analysis

- Understand AWS cloud architecture and best practices
- Analyze Enggo-Backend project
- Design overall system architecture
- Prepare AWS account and IAM roles

### Week 2 – Design & Infrastructure Preparation

- Design Amazon VPC and subnets
- Configure Security Groups
- Create Amazon RDS database
- Prepare development environment

### Week 3 – Development & Containerization

- Develop/update Spring Boot backend
- Configure Spring Data JPA
- Create Dockerfile
- Build Docker Image locally

### Week 4 – AWS Deployment

- Push Docker Image to Amazon ECR
- Create ECS Task Definition
- Deploy to Amazon ECS Fargate
- Configure Application Load Balancer
- Configure Amazon Route 53 & ACM

### Week 5 – CI/CD & Monitoring

- Configure AWS CodeBuild
- Set up CI/CD pipeline
- Configure Amazon CloudWatch
- System-wide testing

---

# 6. Cost Estimation

## Monthly Infrastructure Cost Estimate

| Service | Estimated Cost |
|---------|-----------------|
| Amazon ECS Fargate | ~$5.00 |
| Amazon RDS (db.t3.micro) | ~$15.00 |
| Amazon S3 (Storage & Requests) | ~$1.00 |
| Amazon ECR | ~$0.50 |
| AWS CodeBuild | ~$1.00 |
| Application Load Balancer | ~$16.00 |
| Amazon CloudWatch (Logs & Metrics) | ~$2.00 |
| AWS Data Transfer | ~$5.00 |
| **Total Estimate** | **~$45.50/month** |

### Cost Control Guidelines

- **AWS Budgets:** Automatic alerts when cost exceeds $50 and $100
- **Amazon RDS:** Use db.t3.micro for development, limit backups
- **ECS Fargate:** Optimize resource allocation for CPU/Memory
- **AWS CodeBuild:** Build only when code is pushed to GitHub
- **CloudWatch Logs:** Set retention policy to reduce costs
- **Cleanup after demo:** Delete unused ECS services, RDS instances, ECR images, S3 buckets, ALB

---

# 7. Risk Assessment

## Risk Matrix

| Risk | Impact | Probability |
|------|--------|-------------|
| Amazon ECS deployment failure | High | Medium |
| Amazon RDS connection error | High | Low |
| Amazon ECR upload failure | Medium | Low |
| AWS CodeBuild build failure | Medium | Medium |
| DNS Route 53 configuration error | Medium | Low |
| HTTPS ACM certificate error | Low | Very Low |
| Cost exceeds budget | High | Medium |
| Application performance issues | Medium | Low |

## Mitigation Strategy

- ✅ Enable **Amazon CloudWatch** monitoring for all resources
- ✅ Configure **AWS Budgets** alerts periodically
- ✅ Manage Docker Image versions using **Amazon ECR Lifecycle Policy**
- ✅ Enable **Multi-AZ** for Amazon RDS
- ✅ Apply **IAM policies following least privilege principle**
- ✅ Verify **DNS Route 53** records before deployment
- ✅ Check **ACM certificate status** before enabling HTTPS
- ✅ Backup **Amazon RDS** periodically

## Contingency Plan

- Restore **Docker Image** from Amazon ECR
- Redeploy **ECS Task Definition** from previous version
- Restore **Amazon RDS backup**
- Redeploy via **AWS CodeBuild**
- Reconfigure **DNS Route 53** records if needed
- Reissue **ACM certificate** when validation fails

---

# 8. Expected Results

## Technical Results

The completed project will provide:

✅ **Complete cloud-native backend system**  
✅ **Containerized Spring Boot application**  
✅ **Managed database** on Amazon RDS  
✅ **Automated CI/CD deployment** with GitHub & AWS CodeBuild  
✅ **Container orchestration** with Amazon ECS Fargate  
✅ **Load balancing** with Application Load Balancer  
✅ **Secure HTTPS/TLS communication** with AWS Certificate Manager  
✅ **Domain management** with Amazon Route 53  
✅ **Reliable file storage** with Amazon S3  
✅ **Comprehensive monitoring** with Amazon CloudWatch  
✅ **Secure access control** with AWS IAM  

## Business Value

The project demonstrates real-world deployment of modern backend applications using Spring Boot on AWS managed services, containerization, and DevOps practices.

Cloud-native architecture helps:
- Simplify backend application deployment
- Reduce operational effort
- Improve scalability
- Reduce time for feature development
- Provide reliable platform for expansion

Future improvements may include:
- Integration of **AWS Lambda** for serverless functions
- Use of **Amazon DynamoDB** for NoSQL database
- Integration of **Amazon SQS/SNS** for message queue
- Deployment of **microservices architecture**
- Addition of **caching layer** with Amazon ElastiCache
- Integration of **authentication** with AWS Cognito

