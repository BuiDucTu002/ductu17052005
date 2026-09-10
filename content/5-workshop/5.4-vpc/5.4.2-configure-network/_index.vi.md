---
title: "5.4.2. Cấu hình mạng"
weight: 2
chapter: false
---

# 5.4.2. Cấu hình mạng

Tạo `enggo-igw` cho public subnet, `enggo-nat-gw` tại `enggo-public-subnet-1a`, public route table trỏ `0.0.0.0/0` tới IGW và private route table trỏ `0.0.0.0/0` tới NAT Gateway.

Tạo Security Group theo chuỗi:

```text
Internet -> enggo-alb-sg -> enggo-backend-sg -> enggo-db-sg
```

ALB mở 80/443, backend mở 8080 chỉ từ ALB SG, database mở 3306 chỉ từ backend SG.
