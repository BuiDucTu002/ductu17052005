---
title: "5.10.1. Cấu hình Amazon CloudWatch"
weight: 1
chapter: false
---

# 5.10.1. Cấu hình Amazon CloudWatch

ECS ghi stdout/stderr vào log group `/ecs/enggo-backend`. Tạo alarm cho CPU/Memory ECS trên 80%, RDS FreeStorageSpace dưới 2 GB, DatabaseConnections tăng bất thường, ALB 5XX và TargetResponseTime.

Kiểm tra log sau deploy để phát hiện runtime exception, lỗi JWT và lỗi kết nối database. Gửi alarm qua SNS để có thông báo vận hành.
