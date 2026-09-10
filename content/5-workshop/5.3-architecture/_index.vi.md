---
title: "Kiến trúc hệ thống và dịch vụ AWS"
weight: 3
chapter: true
pre: "<b>5.3. </b>"
date: 2026-09-10
---

# Kiến trúc hệ thống và lựa chọn dịch vụ

## Sơ đồ kiến trúc

```text
Internet / Client App
				|
				| REST API / WebSocket :8080
				v
AWS VPC 10.0.0.0/16
	+-- Public Subnet 10.0.1.0/24
	|     +-- EC2 Ubuntu 22.04 + Elastic IP
	|           +-- enggo-backend (Spring Boot :8080)
	|           +-- my-redis (Redis :6379)
	|
	+-- Private Subnet 10.0.2.0/24
	|     +-- RDS MySQL (database-1 / db.t3.micro)
	|
	+-- Amazon S3 (Audio and User Media)
```

## Danh sách dịch vụ AWS

| Dịch vụ | Vai trò | Lý do lựa chọn |
| --- | --- | --- |
| Amazon VPC | Mạng riêng ảo | Phân tách Public Subnet và Private Subnet. |
| Amazon EC2 | Máy chủ tính toán | Chạy Docker, Spring Boot và Redis. |
| Amazon RDS MySQL | Cơ sở dữ liệu | Có backup tự động và giảm công việc vận hành. |
| AWS ECR | Container Registry | Lưu trữ Docker Image riêng tư và tích hợp IAM. |
| Amazon S3 | Object Storage | Lưu trữ audio bài học và avatar. |
| Security Groups | Firewall ảo | Kiểm soát lưu lượng theo từng tầng. |
| Redis Container | In-memory Cache | Quản lý session, phòng chờ PvP và ELO. |
