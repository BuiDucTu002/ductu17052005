---
title: "5.9. CI/CD"
weight: 9
chapter: false
---

# 5.9. CI/CD với AWS CodeBuild

CodeBuild tự động test, build Docker image, push ECR và tạo `imagedefinitions.json` cho bước deploy ECS.

## `buildspec.yml`

```yaml
version: 0.2
phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
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
  files:
    - imagedefinitions.json
```

## Cấu hình project

**CodeBuild → Create build project**: source GitHub/CodeCommit, image Amazon Linux Standard 5.0, bật **Privileged**, biến `AWS_ACCOUNT_ID`, `AWS_DEFAULT_REGION=ap-northeast-1`, buildspec từ file. CodePipeline nối Source → CodeBuild → ECS Deploy.
