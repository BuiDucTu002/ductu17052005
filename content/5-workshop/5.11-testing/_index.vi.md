---
title: "5.11. Kiểm thử"
weight: 11
chapter: false
---

# 5.11. Kiểm thử hệ thống

## 1. Unit và Integration Test

```bash
mvn test
```

Kiểm tra service, repository, JWT, kết nối schema `tienganh_app` và các endpoint chính.

## 2. End-to-End API qua ALB

```bash
curl -i -X POST http://<ALB-DNS>/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"demo","password":"<MAT_KHAU>"}'

curl -i http://<ALB-DNS>/api/v1/topics
```

Kết quả mong đợi: login trả JWT; topics trả `HTTP 200` và JSON.

## 3. WebSocket và tải

```bash
wscat -c ws://<ALB-DNS>/ws/pvp
```

Kết quả mong đợi: handshake `101 Switching Protocols`, kết nối ổn định khi dùng PvP.

## 4. Failover

Dừng thủ công một ECS Task, kiểm tra ALB loại task không healthy và chuyển traffic sang task còn lại mà không gián đoạn API.
