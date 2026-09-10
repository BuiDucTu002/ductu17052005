---
title: "5.7.1. Cấu hình Load Balancer"
weight: 1
chapter: false
---

# 5.7.1. Cấu hình Load Balancer

Tạo target group `enggo-backend-tg`, target type IP, HTTP/8080, health check `/api/v1/health` hoặc `/actuator/health`. Tạo ALB `enggo-alb` internet-facing trong hai public subnet, gắn `enggo-alb-sg` và forward listener HTTP/80 tới target group. Bật sticky session 86400 giây nếu PvP yêu cầu.
