---
title: "1.5. Tuần 5 worklog"
date: "`r Sys.Date()`"
weight: 5
chapter: false
---

# Tuần 5: Triển khai Hệ thống Enggo-Backend bằng các dịch vụ AWS

### Mục tiêu tuần 5:

- Hiểu và triển khai hàng chục một ứng dụng web backend hoàn chỉnh trên AWS.
- Sử dụng các dịch vụ AWS chủ chốt: EC2, RDS, S3, IAM, và VPC.
- Thiết kế kiến trúc scalable và secure cho ứng dụng.
- Thiết lập CI/CD pipeline và monitoring.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|-----|-----------|--------------|-----------------|------------------|
| 2 | Thiết kế kiến trúc Enggo-Backend. Tạo VPC và subnets. Cấu hình Security Groups. | 17/04/2026 | 17/04/2026 | AWS Architecture |
| 3 | Thiết lập database: RDS MySQL và cấu hình backups. Tạo IAM roles và permissions. | 18/04/2026 | 18/04/2026 | AWS RDS, IAM |
| 4 | Triển khai EC2 instances cho application server. Cài đặt Node.js/Python runtime. Deploy Enggo-Backend application. | 19/04/2026 | 19/04/2026 | AWS EC2, SSH |
| 5 | Cấu hình S3 buckets cho file storage. Tạo CloudFront distribution. Cấu hình CDN. | 20/04/2026 | 20/04/2026 | AWS S3, CloudFront |
| 6 | Thiết lập CloudWatch monitoring và logs. Cấu hình alarms và notifications. Test load balancing. | 21/04/2026 | 21/04/2026 | AWS CloudWatch |
| 7 | Thiết lập CI/CD pipeline với CodePipeline/CodeDeploy. Testing và security validation. | 22/04/2026 | 22/04/2026 | AWS DevOps |

### Kết quả đạt được tuần 5:

**Thiết kế Kiến trúc:**
- Thiết kế đầy đủ kiến trúc cho hệ thống Enggo-Backend
- Tạo VPC và subnets theo bảo mật (security best practices)
- Cấu hình tầng mạng (Network Layer)

**Database Setup:**
- Thiết lập RDS MySQL dành cho ứng dụng
- Cấu hình automated backups và snapshots
- Tạo IAM roles với permissions tối thiểu (least privilege)
- Secure database connections

**Application Deployment:**
- Triển khai EC2 instances cho backend services
- Cài đặt runtime environment (Node.js/Python)
- Deploy Enggo-Backend application
- Cấu hình auto-scaling

**Storage & CDN:**
- Tạo S3 buckets cho static files và uploads
- Cấu hình CloudFront CDN để tăng tốc độ
- Optimize content delivery

**Monitoring & Logging:**
- Thiết lập CloudWatch monitoring
- Cấu hình application logs và metrics
- Tạo alarms cho performance issues
- Real-time notifications

**CI/CD & Testing:**
- Thiết lập automated deployment pipeline
- Cấu hình code deployment automation
- Testing và validation workflows
- Security scanning

**Tổng kết:**
- Triển khai đề cương hoàn chỉnh một ứng dụng web backend trên AWS
- Hiểu về các dịch vụ AWS và cách tất cả chúng của nhân làm việc cùng nhau
- Nắm vứng các best practices cho scalability, security, và performance
- Có kinh nghiệm với infrastructure as code và automation

