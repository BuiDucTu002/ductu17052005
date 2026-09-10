---
title: "Sơ đồ kiến trúc"
weight: 1
chapter: false
pre: "<b>5.3.1. </b>"
date: 2026-09-10
---

# Sơ đồ kiến trúc

```text
Internet / Client App
        |
        | REST API / WebSocket :8080
        v
AWS VPC 10.0.0.0/16
  +-- Public Subnet 10.0.1.0/24
  |     +-- EC2 Ubuntu 22.04 + Elastic IP
  |           +-- enggo-backend (Spring Boot :8080)
  |           +-- my-redis (Redis :6379)
  |
  +-- Private Subnet 10.0.2.0/24
  |     +-- RDS MySQL (database-1 / db.t3.micro)
  |
  +-- Amazon S3 (Audio and User Media)
```
