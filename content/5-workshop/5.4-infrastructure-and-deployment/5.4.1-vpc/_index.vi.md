---
title: "Khởi tạo VPC và Subnets"
weight: 1
chapter: false
pre: "<b>5.4.1. </b>"
date: 2026-09-10
---

# Khởi tạo VPC và Subnets

Vào **VPC Dashboard -> Create VPC**:

* Name tag: `enggo-vpc`.
* IPv4 CIDR: `10.0.0.0/16`.
* Public Subnet: `enggo-public-subnet-1`, `10.0.1.0/24`, AZ `ap-northeast-1a`, bật Auto-assign public IPv4.
* Private Subnet: `enggo-private-subnet-1`, `10.0.2.0/24`, AZ `ap-northeast-1a`.
* Tạo Internet Gateway `enggo-igw` và attach vào VPC.
* Public Route Table: route `0.0.0.0/0` tới `enggo-igw`, associate với Public Subnet.
