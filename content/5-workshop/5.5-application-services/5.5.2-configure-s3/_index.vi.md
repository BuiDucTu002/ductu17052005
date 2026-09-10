---
title: "5.5.2. Cấu hình Amazon S3"
weight: 2
chapter: false
---

# 5.5.2. Cấu hình Amazon S3

Tạo bucket `enggo-media-bucket` ở `ap-northeast-1`, ACLs disabled, Block all public access, Versioning và SSE-S3. Bucket dùng lưu avatar, audio, hình ảnh topic và tài liệu học.

```bash
aws s3api head-bucket --bucket enggo-media-bucket
aws s3api get-bucket-versioning --bucket enggo-media-bucket
aws s3api get-bucket-encryption --bucket enggo-media-bucket
```

Nghiệm thu: kiểm tra được Versioning `Enabled`, encryption `AES256` và không có truy cập công khai.
