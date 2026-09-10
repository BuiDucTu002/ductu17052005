---
title: "5.8. Tên miền và HTTPS"
weight: 8
chapter: false
---

# 5.8. Tên miền và HTTPS

Sau khi ALB hoạt động, dùng Route 53 làm DNS và AWS Certificate Manager (ACM) để mã hóa HTTPS.

## Quy trình

1. Tạo hoặc dùng Hosted Zone trong Route 53.
2. Request certificate ACM cho domain.
3. Chọn DNS validation và tạo record validation.
4. Thêm HTTPS listener port 443 vào ALB.
5. Gắn certificate vào listener và redirect HTTP 80 sang HTTPS 443.

## Kiến trúc truy cập

```text
https://enggo.example.com
        -> Route 53 Alias
        -> ALB :443
        -> enggo-backend-tg :8080
```

Chứng chỉ ACM phải được tạo tại cùng Region với ALB (`ap-northeast-1`).
