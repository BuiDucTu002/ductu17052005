---
title: "5.3. Chuẩn bị dự án"
weight: 3
chapter: false
---

# 5.3. Chuẩn bị dự án

## Kiểm tra mã nguồn

```bash
git clone <ENGGO_BACKEND_REPOSITORY>
cd enggo-backend
mvn clean test
```

Backend dùng Spring Boot 3.4.3, Java 21, Maven và port 8080. Các endpoint chính là `/api/v1/auth`, `/api/v1/topics`, `/api/v1/vocabularies` và WebSocket `/ws/pvp`.

## Quy ước tài nguyên

| Nhóm | Tên |
| --- | --- |
| Region | `ap-northeast-1` |
| VPC | `enggo-vpc` |
| Backend image | `enggo-backend` |
| RDS | `enggo-db` |
| S3 | `enggo-media-bucket` |
| ECS Cluster | `enggo-cluster` |

## Biến môi trường

`DB_HOST`, `DB_PASSWORD`, `SPRING_REDIS_HOST`, `SPRING_REDIS_PORT`, `AWS_S3_BUCKET`, `AWS_REGION`, `JWT_SIGNER_KEY`.
 
![Ảnh của bạn](/ductu17052005/images/5.3.1.png)
![Ảnh của bạn](/ductu17052005/images/5.3.2.png)

## Kết quả mong đợi
Sau khi hoàn thành phần này, bạn sẽ:

+ Tải thành công mã nguồn dự án.
+ Cài đặt đầy đủ các thư viện cần thiết.
+ Cấu hình thành công các biến môi trường.
+ Chạy thành công ứng dụng trên môi trường cục bộ.