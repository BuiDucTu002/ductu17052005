---
title: "5.5.1. Cấu hình Amazon RDS MySQL"
weight: 1
chapter: false
---

# 5.5.1. Cấu hình Amazon RDS MySQL

Tạo DB subnet group `enggo-db-subnet-group` gồm private subnet 1a và 1c. Tạo database `enggo-db`, MySQL 8.0, `db.t3.micro`, gp3 20 GiB, `tienganh_app`, Public access `No`, Security Group `enggo-db-sg`.

```bash
aws rds describe-db-instances --db-instance-identifier enggo-db \
  --query 'DBInstances[0].{Status:DBInstanceStatus,Endpoint:Endpoint.Address,Public:PubliclyAccessible}'
```

Nghiệm thu: trạng thái `available`, có endpoint DNS và `PubliclyAccessible` là `false`.
