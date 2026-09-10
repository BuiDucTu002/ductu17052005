---
title: "1.2. week 2 worklog"
date: 2026-09-10
weight: 2
chapter: false
---

# Week 2: Architecture Design and AWS Infrastructure Preparation

### Week 2 Objectives:

- Design Amazon VPC and subnets
- Configure Security Groups for each layer
- Create Amazon RDS database
- Prepare development environment
- Configure IAM roles for services
- Verify connectivity between components

### Tasks to be accomplished this week:

| No. | Task | Start Date | Completion Date | Resource |
|-----|------|-----------|-----------------|----------|
| 2 | Design VPC architecture and create subnets | 22/04/2026 | 22/04/2026 | AWS VPC Console |
| 3 | Configure Security Groups for ALB, ECS, RDS | 23/04/2026 | 23/04/2026 | AWS Security Groups |
| 4 | Create Amazon RDS database instance | 24/04/2026 | 24/04/2026 | AWS RDS Console |
| 5 | Configure database backups and Multi-AZ | 25/04/2026 | 25/04/2026 | AWS RDS Management |
| 6 | Set up IAM roles for ECS and RDS access | 26/04/2026 | 26/04/2026 | AWS IAM |

### Results achieved in Week 2:

**VPC & Network Architecture:**
- Successfully created Amazon VPC with CIDR block configuration
- Created public and private subnets across multiple availability zones
- Configured Internet Gateway and Route Tables
- Enabled VPC Flow Logs for network monitoring

**Security Configuration:**
- Created Security Groups with appropriate inbound/outbound rules
- Configured ALB Security Group for HTTP/HTTPS traffic (ports 80, 443)
- Configured ECS Security Group with restricted access
- Configured RDS Security Group for database access only from ECS

**Database Setup:**
- Created Amazon RDS MySQL/PostgreSQL database instance
- Configured database size (db.t3.small for production readiness)
- Enabled automated backups with 30-day retention
- Configured Multi-AZ deployment for high availability

**IAM Roles & Policies:**
- Created IAM role for ECS task execution
- Created IAM role for RDS access
- Configured S3 access policies for application
- Set up CloudWatch Logs permissions

**Environment Preparation:**
- Installed AWS CLI v2 and configured credentials
- Set up development environment with required tools
- Tested connectivity to RDS database
- Verified VPC routing and security configuration

### Results achieved in Week 2:

- Successfully created AWS account and configured initial practice environment.

