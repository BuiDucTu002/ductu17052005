---
title: "5.2. Điều kiện chuẩn bị"
weight: 2
chapter: false
---

# 5.1. Chuẩn bị môi trường

Để triển khai EngGo trên AWS tại Region `ap-northeast-1` (Tokyo), cần chuẩn bị tài khoản, quyền truy cập, công cụ local và quy hoạch mạng.

## 1. Tài khoản và quyền IAM

| Hạng mục | Yêu cầu |
| --- | --- |
| AWS Account | Tài khoản hoạt động tại `ap-northeast-1` |
| IAM User/Role | Có quyền thực hiện workshop; production nên dùng least privilege |
| Policy tối thiểu | `AmazonVPCFullAccess`, `AmazonRDSFullAccess`, `AmazonS3FullAccess`, `AmazonEC2ContainerRegistryFullAccess`, `AmazonSSMFullAccess` |

Không đưa Access Key, Secret Key, mật khẩu RDS hoặc JWT signer key vào source code/Git. Với ECS, ưu tiên IAM Role và SSM Parameter Store.

## 2. Công cụ local

- AWS CLI v2, cấu hình bằng `aws configure`.
- Docker và Docker Desktop để build image Spring Boot.
- JDK 21 và Maven để đóng gói file `.jar`.
- Git để quản lý mã nguồn `enggo-backend`.
- Postman để kiểm thử REST API và `wscat` để kiểm thử WebSocket.

```bash
aws configure
aws sts get-caller-identity
aws configure get region
java -version
mvn -version
docker --version
```

Kết quả thành công: `get-caller-identity` trả đúng Account ID/UserId và Region là `ap-northeast-1`.

## 3. Quy hoạch mạng

| Hạng mục | Giá trị |
| --- | --- |
| VPC CIDR | `10.0.0.0/16` |
| Availability Zones | `ap-northeast-1a`, `ap-northeast-1c` |
| Public Subnets | Internet Gateway, ALB, NAT Gateway |
| Private Subnets | ECS/EC2 Backend và RDS MySQL |
