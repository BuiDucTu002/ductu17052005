---
title: "5.10. Giám sát hệ thống"
weight: 10
chapter: false
---

# 5.10. Giám sát với CloudWatch

## CloudWatch Logs

Log group `/ecs/enggo-backend` thu thập stdout/stderr từ ECS Task, hỗ trợ truy vết exception runtime, lỗi kết nối database và lỗi JWT.

## Metrics và Alarms

| Dịch vụ | Metric | Ngưỡng đề xuất |
| --- | --- | --- |
| ECS | CPUUtilization | > 80% trong 5 phút |
| ECS | MemoryUtilization | > 80% trong 5 phút |
| RDS | FreeStorageSpace | < 2 GB |
| RDS | DatabaseConnections | Tăng bất thường |
| ALB | HTTPCode_Target_5XX_Count | Tăng liên tục |
| ALB | TargetResponseTime | Cao hơn SLO |

Tạo alarm trong **CloudWatch → Alarms**, gửi thông báo qua SNS và kiểm tra log sau mỗi lần deploy.
