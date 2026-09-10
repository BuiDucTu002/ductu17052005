---
title: "5.12. Dọn dẹp tài nguyên"
weight: 12
chapter: false
---

# 5.12. Dọn dẹp tài nguyên

Xóa theo thứ tự ngược lại để tránh lỗi dependency:

1. Giảm desired count về `0`, xóa ECS Service và `enggo-cluster`.
2. Xóa `enggo-alb` và target group `enggo-backend-tg`.
3. Xóa RDS `enggo-db` (bỏ final snapshot chỉ trong lab).
4. Làm rỗng, xóa version/object rồi xóa `enggo-media-bucket`.
5. Xóa ElastiCache Redis và S3 VPC Endpoint.
6. Xóa NAT Gateway, release Elastic IP.
7. Xóa route tables, subnets, Security Groups, Internet Gateway.
8. Cuối cùng xóa `enggo-vpc`.

> **Cảnh báo chi phí:** NAT Gateway, Elastic IP, RDS và ALB vẫn tính phí khi còn tồn tại. Kiểm tra Billing sau khi dọn dẹp.
