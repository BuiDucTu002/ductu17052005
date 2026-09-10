---
title: "5.9.1. Cấu hình AWS CodeBuild"
weight: 1
chapter: false
---

# 5.9.1. Cấu hình AWS CodeBuild

CodeBuild dùng source GitHub/CodeCommit, image Amazon Linux Standard 5.0 và bắt buộc bật **Privileged mode** để build Docker. Biến môi trường gồm `AWS_ACCOUNT_ID` và `AWS_DEFAULT_REGION=ap-northeast-1`.

```yaml
version: 0.2
phases:
  pre_build:
    commands:
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/enggo-backend
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_COMMIT_HASH | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:-latest}
  build:
    commands:
      - mvn test
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
  post_build:
    commands:
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - printf '[{"name":"enggo-backend","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
artifacts:
  files: imagedefinitions.json
```
