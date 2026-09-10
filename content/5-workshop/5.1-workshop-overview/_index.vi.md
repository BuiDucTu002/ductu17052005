---
title: "5.1. Tổng quan Workshop"
weight: 1
chapter: false
---

# 5.1. Tổng quan Workshop

## Mục tiêu

Workshop triển khai hệ thống EngGo Backend trên AWS với các lớp mạng, dữ liệu, ứng dụng và vận hành tách biệt.

- Thiết kế VPC public/private theo Availability Zone.
- Chạy Spring Boot container trên ECS Fargate sau Application Load Balancer.
- Kết nối RDS MySQL, S3, Redis và SSM Parameter Store.
- Thiết lập CI/CD, CloudWatch, kiểm thử và dọn dẹp chi phí.

## Kiến trúc tổng thể

{{< mermaid >}}
flowchart LR
  C[Client] --> R53[Route 53]
  R53 --> ALB[ALB HTTPS]
  ALB --> ECS[ECS Fargate]
  ECS --> RDS[(RDS MySQL)]
  ECS --> S3[(S3)]
  ECS --> Redis[(ElastiCache Redis)]
  Git[GitHub] --> Build[CodeBuild]
  Build --> ECR[ECR]
  ECR --> ECS
{{< /mermaid >}}

## Kết quả mong đợi

Các task ECS ở private subnet phục vụ API qua ALB; database không public; log, image và secret được quản lý bằng dịch vụ AWS chuyên dụng.
