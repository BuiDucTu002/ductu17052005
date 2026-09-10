---
title: "5.5.3. Cấu hình Redis"
weight: 3
chapter: false
---

# 5.7. Cấu hình Redis

Redis dùng cho cache và session PvP/real-time. Tạo ElastiCache Redis trong private subnet, gắn `enggo-redis-sg` và chỉ cho phép TCP 6379 từ `enggo-backend-sg`.

## Kết nối ứng dụng

```yaml
spring:
  data:
    redis:
      host: ${SPRING_REDIS_HOST}
      port: ${SPRING_REDIS_PORT:6379}
```

Trong Task Definition:

```json
[
  {"name": "SPRING_REDIS_HOST", "value": "enggo-redis.xxxx.ng.0001.apne1.cache.amazonaws.com"},
  {"name": "SPRING_REDIS_PORT", "value": "6379"}
]
```

**Nghiệm thu:** endpoint Redis phân giải được từ private subnet, SG không mở 6379 ra Internet và backend kết nối thành công.
