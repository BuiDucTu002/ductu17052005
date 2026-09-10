---
title: "Troubleshooting"
weight: 2
chapter: false
pre: "<b>5.5.2. </b>"
date: 2026-09-10
---

# Troubleshooting

**Communications link failure:** Kiểm tra RDS Security Group có Inbound TCP `3306` với source là `enggo-ec2-sg` hay chưa.

**Permission denied khi pull từ ECR:** Đăng nhập ECR trên EC2 bằng `aws ecr get-login-password` trước khi chạy `docker pull` và kiểm tra quyền IAM.
