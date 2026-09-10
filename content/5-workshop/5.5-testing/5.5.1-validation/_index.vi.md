---
title: "Kiểm thử kết quả"
weight: 1
chapter: false
pre: "<b>5.5.1. </b>"
date: 2026-09-10
---

# Kiểm thử kết quả

* **REST API Health:** Gửi `GET http://<EC2_ELASTIC_IP>:8080/api/v1/health` và kiểm tra HTTP 200.
* **WebSocket PvP:** Kết nối `ws://<EC2_ELASTIC_IP>:8080/ws-enggo`, kiểm tra handshake HTTP 101.
* **S3 Media:** Upload audio và kiểm tra URL `https://<bucket>.s3.ap-northeast-1.amazonaws.com/audio/...`.
* **Container logs:**

```bash
docker logs -f enggo-app
```

Kết quả mong đợi là `Started EngGoApplication`, HikariCP tới RDS MySQL và Redis kết nối thành công.
