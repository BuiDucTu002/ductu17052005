---
title: "5.7.2. Triển khai ứng dụng lên Amazon ECS"
weight: 2
chapter: false
---

# 5.7.2. Triển khai ứng dụng lên Amazon ECS

Đăng ký Task Definition Fargate `enggo-backend-task`, port 8080, CPU 512, memory 1024, desired count 2, private subnet và `assignPublicIp=DISABLED`. Gắn service với target group ALB và CloudWatch log group `/ecs/enggo-backend`.
