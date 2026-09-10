---
title: "1.4. Tuần 4 worklog"
date: "`r Sys.Date()`"
weight: 4
chapter: false
---

# Tuần 4: Triển khai trên AWS

### Mục tiêu tuần 4:

- Đẩy Docker Image lên Amazon ECR
- Tạo ECS Task Definition
- Triển khai trên Amazon ECS Fargate
- Cấu hình Application Load Balancer
- Cấu hình Amazon Route 53 và ACM
- Xác thực triển khai

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|------|-----------|--------------|----------------|-------------------|
| 2 | Tạo ECR repository và đẩy Docker Image | 02/05/2026 | 02/05/2026 | AWS ECR |
| 3 | Tạo ECS cluster và task definition | 03/05/2026 | 03/05/2026 | AWS ECS Console |
| 4 | Triển khai lên ECS Fargate | 04/05/2026 | 04/05/2026 | AWS ECS Management |
| 5 | Cấu hình Application Load Balancer | 05/05/2026 | 05/05/2026 | AWS ALB Console |
| 6 | Cấu hình Route 53 DNS và ACM certificates | 06/05/2026 | 06/05/2026 | AWS Route 53 & ACM |

### Kết quả đạt được tuần 4:

**Amazon ECR:**
- Tạo thành công ECR repository cho Enggo-Backend
- Đăng nhập ECR registry từ AWS CLI
- Tag Docker Image với repository URL
- Đẩy Docker Image lên ECR (thành công)
- Xác thực image availability trong ECR console

**ECS Cluster & Task Definition:**
- Tạo ECS cluster cho Enggo-Backend
- Tạo ECS task definition với:
  - Container image từ ECR
  - CPU: 256 units, Memory: 512 MB
  - Environment variables cho RDS connection
  - Logging đến CloudWatch
- Cấu hình task execution role với appropriate permissions

**ECS Fargate Deployment:**
- Triển khai thành công ECS service trên Fargate
- Cấu hình desired task count (2 cho high availability)
- Thiết lập automatic task replacement
- Xác thực tasks chạy trên Fargate

**Application Load Balancer:**
- Tạo Application Load Balancer
- Cấu hình target group với health check
- Thiết lập HTTP/HTTPS listeners
- Xác thực ALB routing traffic đến ECS tasks

**DNS & HTTPS:**
- Mua/cấu hình domain name với Route 53
- Tạo Route 53 alias record trỏ đến ALB
- Yêu cầu ACM certificate cho HTTPS
- Cấu hình ALB listener cho HTTPS với ACM certificate
- Xác thực SSL/TLS certificate valid

