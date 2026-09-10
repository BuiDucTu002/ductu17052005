---
title: "5. Workshop - Triển khai EngGo trên AWS"
date: "`r Sys.Date()`"
weight: 5
chapter: true
---

# Workshop: Triển khai EngGo trên AWS

Workshop hướng dẫn xây dựng và triển khai backend Spring Boot của hệ thống EngGo trên AWS theo kiến trúc cloud-native, bảo mật và có khả năng mở rộng.

## Lộ trình workshop

1. [5.1. Tổng quan Workshop](5.1-workshop-overview/)
2. [5.2. Điều kiện chuẩn bị](5.2-prerequisite/)
3. [5.3. Chuẩn bị dự án](5.3-project-foundation/)
4. [5.4. Hạ tầng mạng](5.4-vpc/)
5. [5.5. Dịch vụ ứng dụng](5.5-application-services/)
6. [5.6. Đóng gói ứng dụng](5.6-containerization/)
7. [5.7. Triển khai ứng dụng](5.7-deploy-application/)
8. [5.8. Tên miền và HTTPS](5.8-domain-and-https/)
9. [5.9. CI/CD](5.9-ci-cd/)
10. [5.10. Giám sát hệ thống](5.10-monitoring/)
11. [5.11. Kiểm thử](5.11-testing/)
12. [5.12. Dọn dẹp tài nguyên](5.12-cleanup/)

## Kiến trúc tổng thể

{{< mermaid >}}
flowchart LR
    U[Client] --> ALB[Application Load Balancer]
    ALB --> ECS[ECS Fargate]
    ECS --> RDS[(RDS MySQL)]
    ECS --> S3[(S3 Media)]
    ECS --> REDIS[(ElastiCache Redis)]
    CODE[GitHub] --> BUILD[CodeBuild]
    BUILD --> ECR[Amazon ECR]
    ECR --> ECS
{{< /mermaid >}}

## Kết quả đầu ra

- Backend EngGo chạy trên ECS Fargate.
- Database RDS MySQL nằm trong private subnet.
- ALB phân phối request và kiểm tra health check.
- Media được lưu trữ bảo mật trên S3.
- Có quy trình CI/CD, logging và monitoring.
