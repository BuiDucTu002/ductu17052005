---
title: "5.8.1. Cấu hình Route 53 và ACM"
weight: 1
chapter: false
---

# 5.8.1. Cấu hình Route 53 và ACM

Trong Route 53 tạo Hosted Zone, request certificate tại ACM, chọn DNS validation và chờ trạng thái `Issued`. Tại ALB tạo listener HTTPS 443, chọn certificate, forward tới `enggo-backend-tg`; listener HTTP 80 dùng redirect tới HTTPS.

```bash
curl -I https://<DOMAIN>
```

Kết quả mong đợi: HTTP request được chuyển sang HTTPS và certificate hợp lệ.
