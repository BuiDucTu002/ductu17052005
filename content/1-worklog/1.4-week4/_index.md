---
title: "1.4. week 4 worklog"
date: 2026-09-10
weight: 4
chapter: false
---

# Week 4: AWS Deployment

### Week 4 Objectives:

- Push Docker Image to Amazon ECR
- Create ECS Task Definition
- Deploy to Amazon ECS Fargate
- Configure Application Load Balancer
- Configure Amazon Route 53 and ACM
- Verify deployment

### Tasks to be accomplished this week:

| No. | Task | Start Date | Completion Date | Resource |
|-----|------|-----------|-----------------|----------|
| 2 | Create ECR repository and push Docker Image | 02/05/2026 | 02/05/2026 | AWS ECR |
| 3 | Create ECS cluster and task definition | 03/05/2026 | 03/05/2026 | AWS ECS Console |
| 4 | Deploy to ECS Fargate | 04/05/2026 | 04/05/2026 | AWS ECS Management |
| 5 | Configure Application Load Balancer | 05/05/2026 | 05/05/2026 | AWS ALB Console |
| 6 | Configure Route 53 DNS and ACM certificates | 06/05/2026 | 06/05/2026 | AWS Route 53 & ACM |

### Results achieved in Week 4:

**Amazon ECR:**
- Successfully created ECR repository for Enggo-Backend
- Logged in to ECR registry from AWS CLI
- Tagged Docker Image with repository URL
- Pushed Docker Image to ECR (successful)
- Verified image availability in ECR console

**ECS Cluster & Task Definition:**
- Created ECS cluster for Enggo-Backend
- Created ECS task definition with:
  - Container image from ECR
  - CPU: 256 units, Memory: 512 MB
  - Environment variables for RDS connection
  - Logging to CloudWatch
- Configured task execution role with appropriate permissions

**ECS Fargate Deployment:**
- Successfully launched ECS service on Fargate
- Configured desired task count (2 for high availability)
- Set up automatic task replacement
- Verified tasks are running in Fargate

**Application Load Balancer:**
- Created Application Load Balancer
- Configured target group with health check
- Set up HTTP/HTTPS listeners
- Verified ALB is routing traffic to ECS tasks

**DNS & HTTPS:**
- Purchased/configured domain name with Route 53
- Created Route 53 alias record pointing to ALB
- Requested ACM certificate for HTTPS
- Configured ALB listener for HTTPS with ACM certificate
- Verified SSL/TLS certificate is valid

