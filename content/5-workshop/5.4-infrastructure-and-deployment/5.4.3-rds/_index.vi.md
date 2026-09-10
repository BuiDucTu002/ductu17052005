---
title: "Khởi tạo RDS MySQL"
weight: 3
chapter: false
pre: "<b>5.4.3. </b>"
date: 2026-09-10
---

# Khởi tạo RDS MySQL

Vào **RDS Console -> Create database**:

* Engine: MySQL Community Edition.
* Template: Free Tier.
* Identifier: `database-1`.
* Username: `admin`.
* Instance class: `db.t3.micro`.
* VPC: `enggo-vpc`.
* Public access: **No**.
* Security group: `enggo-rds-sg`.

Lưu lại RDS Endpoint sau khi database được tạo.
