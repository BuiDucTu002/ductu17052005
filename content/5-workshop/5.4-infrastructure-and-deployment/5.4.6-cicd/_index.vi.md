---
title: "GitHub Actions và ECR"
weight: 6
chapter: false
pre: "<b>5.4.6. </b>"
date: 2026-09-10
---

# GitHub Actions và ECR

Tạo `.github/workflows/deploy.yml` để build Docker Image, push lên ECR và deploy qua SSH tới EC2.

```yaml
name: EngGo Backend CI/CD Pipeline

on:
  push:
    branches: ["main"]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1
      - uses: aws-actions/amazon-ecr-login@v2
        id: login-ecr
      - name: Build and push
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: enggo-backend
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

  deploy-to-ec2:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.ap-northeast-1.amazonaws.com
            docker stop enggo-app || true
            docker rm enggo-app || true
            docker pull ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest
            docker run -d --name enggo-app -p 8080:8080 --link my-redis:redis ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest
```

Secrets cần khai báo: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_ACCOUNT_ID`, `EC2_HOST`, `EC2_SSH_KEY`, `RDS_ENDPOINT`, `DB_USER`, `DB_PASSWORD`, `S3_BUCKET_NAME`.
