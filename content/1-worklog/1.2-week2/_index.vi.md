---
title: "1.2. Tuần 2 worklog"
date: 2026-09-10
weight: 2
chapter: false
---

# Tuần 2: Thiết kế Kiến trúc và Chuẩn bị Hạ tầng AWS

### Mục tiêu tuần 2:

- Thiết kế Amazon VPC và các subnets
- Cấu hình Security Groups cho mỗi layer
- Tạo Amazon RDS database
- Chuẩn bị môi trường phát triển
- Cấu hình IAM roles cho services
- Xác thực kết nối giữa các thành phần

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|------|-----------|--------------|----------------|-------------------|
| 2 | Thiết kế kiến trúc VPC và tạo subnets | 22/04/2026 | 22/04/2026 | AWS VPC Console |
| 3 | Cấu hình Security Groups cho ALB, ECS, RDS | 23/04/2026 | 23/04/2026 | AWS Security Groups |
| 4 | Tạo Amazon RDS database instance | 24/04/2026 | 24/04/2026 | AWS RDS Console |
| 5 | Cấu hình automated backups và Multi-AZ | 25/04/2026 | 25/04/2026 | AWS RDS Management |
| 6 | Thiết lập IAM roles cho ECS và RDS access | 26/04/2026 | 26/04/2026 | AWS IAM |

### Kết quả đạt được tuần 2:

**VPC & Network Architecture:**
- Tạo thành công Amazon VPC với cấu hình CIDR block
- Tạo public và private subnets trên multiple availability zones
- Cấu hình Internet Gateway và Route Tables
- Bật VPC Flow Logs để monitor network

**Cấu hình Security:**
- Tạo Security Groups với inbound/outbound rules phù hợp
- Cấu hình ALB Security Group cho HTTP/HTTPS traffic (ports 80, 443)
- Cấu hình ECS Security Group với restricted access
- Cấu hình RDS Security Group chỉ cho phép từ ECS

**Thiết lập Database:**
- Tạo Amazon RDS MySQL/PostgreSQL database instance
- Cấu hình database size (db.t3.small for production readiness)
- Bật automated backups với retention 30 days
- Cấu hình Multi-AZ deployment để đảm bảo high availability

**IAM Roles & Policies:**
- Tạo IAM role cho ECS task execution
- Tạo IAM role cho RDS access
- Cấu hình S3 access policies cho application
- Thiết lập CloudWatch Logs permissions

**Chuẩn bị Environment:**
- Cài đặt AWS CLI v2 và cấu hình credentials
- Thiết lập development environment với các công cụ cần thiết
- Kiểm tra kết nối đến RDS database
- Xác thực VPC routing và security configuration

