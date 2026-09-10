---
title: "5.6.2. Đẩy Image lên Amazon ECR"
weight: 2
chapter: false
---

# 5.6.2. Đẩy Image lên Amazon ECR

```bash
aws ecr create-repository --repository-name enggo-backend --region ap-northeast-1
aws ecr get-login-password --region ap-northeast-1 |
  docker login --username AWS --password-stdin \
  YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com
docker tag enggo-backend:latest \
  YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest
docker push YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest
```

**Nghiệm thu:** image hiển thị trong repository `enggo-backend` trên ECR.
