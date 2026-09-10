---
title: "CI/CD với GitHub Actions và ECR"
weight: 6
chapter: false
pre: "<b>5.4.6. </b>"
date: 2026-09-10
---


### CI/CD với GitHub Actions và ECR

Để tối ưu hóa quy trình phát triển và triển khai phần mềm, dự án **EngGo** tích hợp quy trình **CI/CD (Continuous Integration / Continuous Deployment)** tự động hóa bằng **GitHub Actions**. 

Mỗi khi có thao tác đẩy mã nguồn (`push`) hoặc gộp nhánh (`merge pull request`) vào nhánh `main`, hệ thống tự động biên dịch, đóng gói Docker Image, đẩy lên **Amazon ECR** và cập nhật phiên bản mới lên máy chủ **Amazon EC2**.

![Sơ đồ luồng tự động hóa CI/CD với GitHub Actions, ECR và EC2](IMAGES/cicd-pipeline-architecture.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh sơ đồ thể hiện luồng: Developer -> Push Code GitHub -> Trigger GitHub Actions -> Build & Push ECR -> SSH Deploy EC2.*

---

#### 1. Cấu hình GitHub Secrets (Bảo mật thông tin đăng nhập)

Để GitHub Actions có quyền tương tác với tài nguyên AWS và truy cập máy chủ EC2, các thông tin nhạy cảm được lưu trữ an toàn tại mục **Settings $\rightarrow$ Secrets and variables $\rightarrow$ Actions** của GitHub Repository:

| Tên Secret | Mô tả & Giá trị |
| :--- | :--- |
| **`AWS_ACCESS_KEY_ID`** | Access Key ID của IAM User có quyền truy cập ECR (`enggo-s3-user` hoặc admin) |
| **`AWS_SECRET_ACCESS_KEY`** | Secret Access Key tương ứng |
| **`EC2_HOST`** | Địa chỉ Public IP của máy chủ EC2 (`enggo-backend-server`) |
| **`EC2_USERNAME`** | Tên người dùng hệ điều hành EC2 (Ví dụ: `ubuntu`) |
| **`EC2_SSH_KEY`** | Toàn bộ nội dung file khóa riêng tư SSH (`enggo-key.pem`) |

![Cấu hình Secrets trên GitHub Repository](IMAGES/step1-github-secrets.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình trang Secrets and variables trong GitHub Settings hiển thị danh sách các biến AWS và EC2.*

---

#### 2. Xây dựng Kịch bản Tự động hóa (`.github/workflows/deploy.yml`)

Khởi tạo tệp cấu hình `.github/workflows/deploy.yml` tại thư mục gốc của dự án `enggo-backend`:

```yaml
name: EngGo Backend CI/CD Pipeline

on:
  push:
    branches: [ "main" ]

env:
  AWS_REGION: ap-northeast-1
  ECR_REPOSITORY: enggo-backend
  CONTAINER_NAME: enggo-backend-app

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    # 1. Checkout mã nguồn từ GitHub
    - name: Checkout Code
      uses: actions/checkout@v3

    # 2. Thiết lập môi trường Java 17
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    # 3. Biên dịch và kiểm thử ứng dụng
    - name: Build with Maven
      run: mvn clean package -DskipTests

    # 4. Xác thực với AWS Credentials
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    # 5. Đăng nhập vào Amazon ECR
    - name: Log in to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    # 6. Đóng gói Docker Image và Đẩy lên ECR
    - name: Build, Tag, and Push Image to ECR
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        # Build image và gắn tag theo Commit SHA và latest
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        # Push cả 2 tags lên Amazon ECR
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    # 7. Triển khai tự động lên máy chủ EC2 qua SSH
    - name: Deploy to EC2 via SSH
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USERNAME }}
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          # Đăng nhập vào ECR trên EC2
          aws ecr get-login-password --region ${{ env.AWS_REGION }} \vert{} docker login --username AWS --password-stdin${{ steps.login-ecr.outputs.registry }}
          
          # Dừng và xóa Container cũ nếu đang chạy
          docker stop ${{ env.CONTAINER_NAME }} || true
          docker rm ${{ env.CONTAINER_NAME }} || true
          
          # Kéo Image mới nhất về
          docker pull ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:latest
          
          # Khởi chạy Container mới
          docker run -d \
            --name ${{ env.CONTAINER_NAME }} \
            -p 8080:8080 \
            -e SPRING_DATASOURCE_URL="jdbc:mysql://[enggo-db-instance.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com:3306/enggo_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC](https://enggo-db-instance.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com:3306/enggo_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC)" \
            -e SPRING_DATASOURCE_USERNAME="admin" \
            -e SPRING_DATASOURCE_PASSWORD="EngGoPass123!" \
            --restart always \
            ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:latest
            
          # Dọn dẹp các Docker Image thừa để giải phóng dung lượng ổ cứng
          docker image prune -f

```

---

#### 3. Kết quả Thực thi Pipeline

Mỗi lần đẩy mã nguồn lên nhánh `main`, hệ thống kích hoạt Workflow tự động. Tiến trình thực thi được theo dõi trực tiếp tại tab **Actions** trên GitHub Repository.


*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình giao diện GitHub Actions hiển thị tất cả các bước trong pipeline báo tích xanh (Success).*

#### Đánh giá Hiệu quả Tự động hóa:

* **Tốc độ Triển khai:** Rút ngắn thời gian Release phiên bản mới từ 15-20 phút (thao tác thủ công) xuống chỉ còn **2-3 phút**.
* **Độ Tin cậy:** Loại bỏ các lỗi do thao tác con người (Human Error) trong quá trình Build/Deploy.
* **Đồng bộ hóa Version:** Mọi phiên bản build đều được đánh vết chính xác theo Commit Hash (`github.sha`) trên Amazon ECR, giúp rollback phiên bản tức thì khi gặp sự cố.

