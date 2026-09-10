---
title: "5.4. Hạ tầng mạng"
weight: 4
chapter: false
---

# 5.2. Thiết kế và triển khai VPC

VPC riêng giúp cô lập hạ tầng EngGo, kiểm soát lưu lượng giữa ALB, Backend và Database.

## 5.2.1. Khởi tạo VPC

| Thông số | Giá trị |
| --- | --- |
| VPC Name | `enggo-vpc` |
| IPv4 CIDR | `10.0.0.0/16` |
| IPv6 | No IPv6 CIDR block |
| Tenancy | Default |
| DNS Hostnames | Enabled |
| DNS Resolution | Enabled |

Trên Console: **VPC → Your VPCs → Create VPC → VPC only**, nhập `enggo-vpc` và `10.0.0.0/16`. Sau đó vào **Actions → Edit VPC settings**, bật DNS hostnames và DNS resolution.

```bash
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=enggo-vpc}]' \
  --query 'Vpc.VpcId' --output text)
aws ec2 modify-vpc-attribute --vpc-id "$VPC_ID" --enable-dns-hostnames '{"Value":true}'
aws ec2 modify-vpc-attribute --vpc-id "$VPC_ID" --enable-dns-support '{"Value":true}'
```

**Nghiệm thu:** VPC `Available`, CIDR đúng và hai DNS settings là `Enabled`.

## 5.2.2. Subnet

| Subnet | CIDR | AZ | Loại |
| --- | --- | --- | --- |
| `enggo-public-subnet-1a` | `10.0.1.0/24` | `ap-northeast-1a` | Public |
| `enggo-public-subnet-1c` | `10.0.2.0/24` | `ap-northeast-1c` | Public |
| `enggo-private-subnet-1a` | `10.0.11.0/24` | `ap-northeast-1a` | Private |
| `enggo-private-subnet-1c` | `10.0.12.0/24` | `ap-northeast-1c` | Private |

Public subnet dành cho ALB/NAT Gateway; private subnet dành cho ECS và RDS.

## 5.2.3. Internet Gateway

Tạo `enggo-igw` trên **VPC → Internet gateways**, sau đó **Actions → Attach to VPC → enggo-vpc**.

```bash
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=enggo-igw}]' \
  --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --vpc-id "$VPC_ID" --internet-gateway-id "$IGW_ID"
```

**Nghiệm thu:** `enggo-igw` chuyển từ `Unattached` sang `Attached` và gắn đúng VPC.

## 5.2.4. NAT Gateway

NAT Gateway cho phép tài nguyên private đi ra Internet nhưng không nhận kết nối vào trực tiếp.

1. Vào **VPC → NAT gateways → Create NAT gateway**.
2. Name: `enggo-nat-gw`.
3. Subnet: `enggo-public-subnet-1a`.
4. Connectivity type: `Public`.
5. Chọn **Allocate Elastic IP** rồi tạo NAT Gateway.

```bash
EIP_ALLOC_ID=$(aws ec2 allocate-address --domain vpc \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=enggo-nat-eip}]' \
  --query 'AllocationId' --output text)
NAT_GW_ID=$(aws ec2 create-nat-gateway --subnet-id "$SUBNET_PUB_1A" \
  --allocation-id "$EIP_ALLOC_ID" \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=enggo-nat-gw}]' \
  --query 'NatGateway.NatGatewayId' --output text)
```

NAT Gateway cần 1–3 phút để chuyển sang `Available`. **Nghiệm thu:** có Elastic IP và nằm trong public subnet 1a.

## 5.2.5. Route Table

| Route table | Default route | Subnet association |
| --- | --- | --- |
| `enggo-public-rt` | `0.0.0.0/0 → enggo-igw` | Public 1a, 1c |
| `enggo-private-rt` | `0.0.0.0/0 → enggo-nat-gw` | Private 1a, 1c |

```bash
RT_PUB_ID=$(aws ec2 create-route-table --vpc-id "$VPC_ID" \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=enggo-public-rt}]' \
  --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id "$RT_PUB_ID" \
  --destination-cidr-block 0.0.0.0/0 --gateway-id "$IGW_ID"
aws ec2 associate-route-table --subnet-id "$SUBNET_PUB_1A" --route-table-id "$RT_PUB_ID"
aws ec2 associate-route-table --subnet-id "$SUBNET_PUB_1C" --route-table-id "$RT_PUB_ID"

RT_PRI_ID=$(aws ec2 create-route-table --vpc-id "$VPC_ID" \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=enggo-private-rt}]' \
  --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id "$RT_PRI_ID" \
  --destination-cidr-block 0.0.0.0/0 --nat-gateway-id "$NAT_GW_ID"
aws ec2 associate-route-table --subnet-id "$SUBNET_PRI_1A" --route-table-id "$RT_PRI_ID"
aws ec2 associate-route-table --subnet-id "$SUBNET_PRI_1C" --route-table-id "$RT_PRI_ID"
```

**Nghiệm thu:** public route đi qua IGW; private route đi qua NAT Gateway.

## 5.2.6. Security Group

| Security Group | Inbound |
| --- | --- |
| `enggo-alb-sg` | TCP 80/443 từ `0.0.0.0/0` |
| `enggo-backend-sg` | TCP 8080 từ `enggo-alb-sg` |
| `enggo-db-sg` | TCP 3306 từ `enggo-backend-sg` |

```bash
ALB_SG_ID=$(aws ec2 create-security-group --group-name enggo-alb-sg \
  --description "Security group for ALB" --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id "$ALB_SG_ID" \
  --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id "$ALB_SG_ID" \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

BACKEND_SG_ID=$(aws ec2 create-security-group --group-name enggo-backend-sg \
  --description "Security group for Spring Boot ECS" --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id "$BACKEND_SG_ID" \
  --protocol tcp --port 8080 --source-group "$ALB_SG_ID"

DB_SG_ID=$(aws ec2 create-security-group --group-name enggo-db-sg \
  --description "Security group for RDS MySQL" --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id "$DB_SG_ID" \
  --protocol tcp --port 3306 --source-group "$BACKEND_SG_ID"
```

**Nghiệm thu:** chuỗi bảo mật `Internet → ALB SG → Backend SG → Database SG`.

## 5.2.7. Sơ đồ lưu lượng

{{< mermaid >}}
flowchart LR
  Internet --> ALB[enggo-alb]
  ALB --> Backend[ECS Backend :8080]
  Backend --> RDS[RDS MySQL :3306]
  Backend --> NAT[NAT Gateway]
  NAT --> IGW[Internet Gateway]
{{< /mermaid >}}
