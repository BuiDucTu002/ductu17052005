---
title: "5.4.3. Cấu hình S3 VPC Endpoint"
weight: 8
chapter: false
---

# 5.8. Cấu hình S3 VPC Endpoint

Gateway Endpoint giúp private subnet truy cập S3 qua mạng nội bộ AWS, giảm phụ thuộc NAT Gateway và không phát sinh phí giờ chạy endpoint.

| Thông số | Giá trị |
| --- | --- |
| Name | `enggo-s3-endpoint` |
| Service | `com.amazonaws.ap-northeast-1.s3` |
| Type | Gateway |
| VPC | `enggo-vpc` |
| Route table | `enggo-private-rt` |

Trên Console: **VPC → Endpoints → Create endpoint**, chọn AWS services, S3 loại Gateway, VPC `enggo-vpc`, route table `enggo-private-rt`, policy Full Access hoặc policy giới hạn bucket.

```bash
S3_ENDPOINT_ID=$(aws ec2 create-vpc-endpoint \
  --vpc-id "$VPC_ID" \
  --service-name com.amazonaws.ap-northeast-1.s3 \
  --route-table-ids "$RT_PRI_ID" \
  --tag-specifications 'ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=enggo-s3-endpoint}]' \
  --query 'VpcEndpoint.VpcEndpointId' --output text)
```

**Nghiệm thu:** endpoint `Available`; private route table có route tới Prefix List S3 qua `vpce-xxxx`.
