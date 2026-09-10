---
title: "Chuẩn bị tài khoản và môi trường"
weight: 1
chapter: false
pre: "<b>5.2.1. </b>"
date: 2026-09-10
---

# Chuẩn bị tài khoản và môi trường

1. Tài khoản AWS Admin hoặc IAM User có quyền tạo VPC, EC2, RDS, ECR, S3 và Security Groups.
2. Region sử dụng: `ap-northeast-1` (Tokyo).
3. Môi trường local: Git, JDK 21, Maven, Docker Engine, SSH Client và MySQL Workbench.
4. Repository `enggo-backend` có `Dockerfile` và GitHub Actions Workflow.

{{% notice warning %}}
Chọn instance `t3.micro` cho EC2/RDS và xóa tài nguyên sau khi hoàn thành để tránh phát sinh chi phí.
{{% /notice %}}
