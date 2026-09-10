---
title: "5.5.4. IAM và Parameter Store"
weight: 4
chapter: false
---

# 5.6. Chuẩn bị IAM và Parameter Store

Tách quyền thành `ecsTaskExecutionRole` (pull image, ghi CloudWatch Logs) và `enggoTaskRole` (S3, SSM, các dịch vụ runtime). Không lưu secret trong Docker image.

## Các parameter cần tạo

| Parameter | Loại | Mục đích |
| --- | --- | --- |
| `/config/enggo-backend/DB_PASSWORD` | SecureString | Mật khẩu RDS |
| `/config/enggo-backend/JWT_SIGNER_KEY` | SecureString | Ký JWT |
| `/config/enggo-backend/AWS_S3_ACCESS_KEY` | SecureString | Tương thích cấu hình ứng dụng |
| `/config/enggo-backend/AWS_S3_SECRET_KEY` | SecureString | Tương thích cấu hình ứng dụng |

```bash
aws ssm put-parameter --name /config/enggo-backend/DB_PASSWORD \
  --type SecureString --value "<MAT_KHAU>" --overwrite --region ap-northeast-1
aws ssm put-parameter --name /config/enggo-backend/JWT_SIGNER_KEY \
  --type SecureString --value "<JWT_KEY>" --overwrite --region ap-northeast-1
```

Trong Task Definition, ánh xạ các parameter vào `secrets`. Task Execution Role phải có `ssm:GetParameters`, `kms:Decrypt` (nếu dùng CMK) và quyền ghi CloudWatch Logs.
