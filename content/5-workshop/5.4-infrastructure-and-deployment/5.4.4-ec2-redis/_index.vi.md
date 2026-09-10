---
title: "Khởi tạo EC2 và Redis"
weight: 4
chapter: false
pre: "<b>5.4.4. </b>"
date: 2026-09-10
---

# Khởi tạo EC2 và Redis

Tạo EC2 Ubuntu Server 22.04 LTS, instance `t3.micro`, Public Subnet và Security Group `enggo-ec2-sg`. Gán Elastic IP.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose-v2 -y
sudo usermod -aG docker ubuntu
newgrp docker

docker run -d --name my-redis -p 6379:6379 --restart always redis:alpine
```
