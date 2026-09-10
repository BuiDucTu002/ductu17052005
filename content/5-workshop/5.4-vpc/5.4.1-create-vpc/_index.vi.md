---
title: "5.4.1. Tạo VPC"
weight: 1
chapter: false
---

# 5.4.1. Tạo VPC

Tạo VPC `enggo-vpc` với CIDR `10.0.0.0/16`, bật DNS hostnames và DNS resolution. Tạo bốn subnet tại `ap-northeast-1a` và `ap-northeast-1c` theo bảng quy hoạch của trang Hạ tầng mạng.

```bash
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=enggo-vpc}]' \
  --query 'Vpc.VpcId' --output text)
aws ec2 modify-vpc-attribute --vpc-id "$VPC_ID" --enable-dns-hostnames '{"Value":true}'
aws ec2 modify-vpc-attribute --vpc-id "$VPC_ID" --enable-dns-support '{"Value":true}'
```

Nghiệm thu: VPC `Available`, CIDR chính xác và hai DNS settings là `Enabled`.
