---
title: "5.5. Dịch vụ ứng dụng"
weight: 5
chapter: false
---

# 5.3. Triển khai các dịch vụ lưu trữ

## 5.3.1. Amazon S3 Bucket

S3 lưu avatar, file âm thanh, hình ảnh chủ đề và tài liệu học tập.

| Thông số | Giá trị |
| --- | --- |
| Bucket | `enggo-media-bucket` |
| Region | `ap-northeast-1` |
| Object Ownership | ACLs disabled |
| Public access | Block all public access |
| Versioning | Enabled |
| Encryption | SSE-S3 / AES-256 |

Trên Console: **S3 → Create bucket**, chọn Tokyo, tắt ACL, bật Block Public Access, Versioning và SSE-S3.

```bash
aws s3api create-bucket --bucket enggo-media-bucket --region ap-northeast-1 \
  --create-bucket-configuration LocationConstraint=ap-northeast-1
aws s3api put-bucket-versioning --bucket enggo-media-bucket \
  --versioning-configuration Status=Enabled
aws s3api put-bucket-encryption --bucket enggo-media-bucket \
  --server-side-encryption-configuration \
  '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'
```

CORS trong **Permissions → CORS**:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": ["ETag"]
  }
]
```

Production nên thay `AllowedOrigins: ["*"]` bằng domain frontend cụ thể.

**Nghiệm thu:** bucket ở đúng Region, Block Public Access là On, Versioning và AES-256 đã bật.

## 5.3.2. Amazon RDS MySQL

| Thông số | Giá trị |
| --- | --- |
| Engine | MySQL 8.0.x |
| Instance | `db.t3.micro` / `db.t4g.micro` |
| Storage | gp3, 20 GiB |
| DB subnet group | `enggo-db-subnet-group` |
| Public access | No |
| Security Group | `enggo-db-sg` |
| Database | `tienganh_app` |
| Username | `admin` |

### Tạo DB Subnet Group

Vào **RDS → Subnet groups → Create DB subnet group**:

- Name: `enggo-db-subnet-group`.
- VPC: `enggo-vpc`.
- AZ: `ap-northeast-1a`, `ap-northeast-1c`.
- Subnet: `10.0.11.0/24`, `10.0.12.0/24`.

### Tạo DB Instance

Chọn **Databases → Create database → Standard create → MySQL → Dev/Test**. Đặt identifier `enggo-db`, username `admin`, instance `db.t3.micro`, storage gp3 20 GiB, VPC `enggo-vpc`, subnet group vừa tạo, **Public access: No**, SG `enggo-db-sg`, database name `tienganh_app`.

```bash
aws rds create-db-subnet-group \
  --db-subnet-group-name enggo-db-subnet-group \
  --db-subnet-group-description "Subnet group for EngGo RDS" \
  --subnet-ids "$SUBNET_PRI_1A" "$SUBNET_PRI_1C"
aws rds create-db-instance --db-instance-identifier enggo-db \
  --db-instance-class db.t3.micro --engine mysql \
  --master-username admin --master-user-password "<MAT_KHAU_THUC_TE>" \
  --allocated-storage 20 --no-publicly-accessible \
  --db-subnet-group-name enggo-db-subnet-group \
  --vpc-security-group-ids "$DB_SG_ID" --db-name tienganh_app \
  --region ap-northeast-1
```

**Nghiệm thu:** `enggo-db` là `Available`, có endpoint RDS và `Publicly Accessible: No`.

## 5.3.3. Schema `tienganh_app`

Các phân hệ: User Management, Learning Content, Progress & Analytics và PvP.

```yaml
spring:
  datasource:
    url: jdbc:mysql://${DB_HOST}:3306/tienganh_app?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
    username: admin
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQLDialect
        format_sql: true
```

```sql
USE tienganh_app;

CREATE TABLE IF NOT EXISTS users (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50) NOT NULL UNIQUE,
  email VARCHAR(100) NOT NULL UNIQUE,
  password VARCHAR(255) NOT NULL,
  avatar_url VARCHAR(255),
  elo_rating INT DEFAULT 1000,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS topics (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(100) NOT NULL,
  description TEXT,
  image_url VARCHAR(255)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS vocabularies (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  topic_id BIGINT NOT NULL,
  word VARCHAR(100) NOT NULL,
  phonetic VARCHAR(100),
  meaning TEXT NOT NULL,
  audio_url VARCHAR(255),
  FOREIGN KEY (topic_id) REFERENCES topics(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**Nghiệm thu:** Hibernate kết nối đúng schema, bảng dùng `utf8mb4` và foreign key hoạt động.
