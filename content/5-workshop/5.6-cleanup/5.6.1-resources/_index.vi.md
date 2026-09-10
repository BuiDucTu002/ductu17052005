---
title: "Xóa tài nguyên AWS"
weight: 1
chapter: false
pre: "<b>5.6.1. </b>"
date: 2026-09-10
---

# Xóa tài nguyên AWS

1. Xóa RDS `database-1`, bỏ chọn final snapshot nếu không cần giữ dữ liệu.
2. Terminate EC2 và release Elastic IP.
3. Xóa repository `enggo-backend` và empty S3 bucket `enggo-bucket`.
4. Xóa Security Groups `enggo-rds-sg`, `enggo-ec2-sg` và VPC `enggo-vpc`.

{{% notice warning %}}
Kiểm tra Billing Dashboard sau cleanup để chắc chắn không còn tài nguyên tính phí.
{{% /notice %}}
