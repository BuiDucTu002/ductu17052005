---
title: "Thiết lập Security Groups"
weight: 2
chapter: false
pre: "<b>5.4.2. </b>"
date: 2026-09-10
---

# Thiết lập Security Groups

Tạo `enggo-ec2-sg` với SSH port `22` từ **My IP** và Custom TCP port `8080` từ `0.0.0.0/0` cho API/WebSocket.

Tạo `enggo-rds-sg` với MySQL/Aurora port `3306`, source là Security Group ID của `enggo-ec2-sg`.

{{% notice info %}}
Chỉ máy chủ EC2 được phép kết nối tới database RDS.
{{% /notice %}}
