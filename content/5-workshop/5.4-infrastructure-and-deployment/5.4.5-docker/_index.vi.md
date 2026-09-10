---
title: "Docker Multi-stage Build"
weight: 5
chapter: false
pre: "<b>5.4.5. </b>"
date: 2026-09-10
---

# Docker Multi-stage Build

---

# KHỞI TẠO VÀ CẤU HÌNH AMAZON ECR (CONTAINER REGISTRY)

---

## 1. TỔNG QUAN VỀ AMAZON ECR

Để triển khai ứng dụng Spring Boot Backend theo kiến trúc Microservices/Container hóa lên các dịch vụ tính toán của AWS (như **Amazon ECS** hoặc **AWS Fargate**), việc đóng gói ứng dụng thành các **Docker Image** và lưu trữ tại một kho quản lý an toàn là điều bắt buộc.

**Amazon Elastic Container Registry (ECR)** là dịch vụ kho lưu trữ Container riêng tư (Private Container Registry) do AWS quản lý hoàn toàn. ECR đóng vai trò làm cầu nối trong quy trình CI/CD:

![Mô hình quy trình đóng gói và lưu trữ Docker Image lên Amazon ECR](IMAGES/ecr-architecture-diagram.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh sơ đồ mô tả luồng: Source Code -> Build Docker Image -> Push to Amazon ECR Repository -> Deploy to ECS/EC2.*

### Ưu điểm kỹ thuật của Amazon ECR:
* **Tích hợp bảo mật cao:** Phân quyền truy cập đẩy/rút (Push/Pull) image chặt chẽ thông qua **AWS IAM**.
* **Quét lỗ hổng tự động (Image Scanning):** Tự động phát hiện các lỗ hổng bảo mật trong các thư viện phụ thuộc (Dependencies) và OS base image.
* **Tích hợp tự nhiên với ECS/EKS:** Giúp việc kéo (Pull) Image triển khai lên Container Cluster diễn ra cực nhanh thông qua đường truyền nội bộ AWS.

---

## 2. QUY TRÌNH THỰC HIỆN TRÊN AWS CONSOLE

### Bước 1: Khởi tạo Private Repository (`enggo-backend`)

1. Đăng nhập vào **AWS Management Console** $\rightarrow$ Tìm kiếm và chọn dịch vụ **Elastic Container Registry**.
2. Tại menu bên trái, chọn **Repositories** $\rightarrow$ Nhấn nút **Create repository**.
3. Cấu hình các thông số cơ bản:
   * **Visibility settings:** Chọn **Private** (Chỉ cho phép các tài khoản IAM được ủy quyền truy cập).
   * **Repository name:** Nhập `enggo-backend`.
4. Cấu hình Nâng cao (Image scan & Encryption):
   * **Image scan settings:** Bật nút **Scan on push** (Hệ thống sẽ tự động quét lỗ hổng bảo mật mỗi khi có Image mới được đẩy lên).
   * **KMS encryption:** Giữ nguyên mặc định (`Disabled` - Sử dụng mã hóa mặc định của S3/ECR để tiết kiệm chi phí).
5. Nhấn **Create repository**.

![Tạo mới Private Repository enggo-backend trên Amazon ECR](IMAGES/step1-create-ecr-repo.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình trang khởi tạo Repository với tên enggo-backend và trạng thái Private.*

---

### Bước 2: Thu thập thông tin Repository URI

Sau khi tạo thành công, ECR sẽ cấp một đường dẫn định danh duy nhất cho Repository (**URI**).

1. Tại danh sách **Repositories**, tìm đến kho `enggo-backend`.
2. Sao chép chuỗi tại cột **URI**.
   * *Định dạng URI tiêu chuẩn:* `<AWS_ACCOUNT_ID>.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend`
   * *Ví dụ:* `123456789012.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend`

![Thông tin Repository URI trên giao diện Amazon ECR](IMAGES/step2-ecr-uri.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình danh sách Repositories hiển thị cột URI của enggo-backend.*

---

## 3. THAO TÁC ĐÓNG GÓI VÀ ĐẨY DOCKER IMAGE LÊN ECR

Dưới đây là quy trình thực thi lệnh bên dưới máy cục bộ (hoặc từ máy chủ CI/CD Runner) để đóng gói và đẩy (Push) Docker Image của ứng dụng Spring Boot lên ECR.

### 1. Tạo tệp `Dockerfile` cho ứng dụng Spring Boot

Tại thư mục gốc của dự án `enggo-backend`, khởi tạo tệp `Dockerfile` với nội dung tối ưu dung lượng (Multi-stage build):

```dockerfile
# Stage 1: Build file JAR với Maven
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Chạy ứng dụng với OpenJDK Runtime
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]

```

---

### 2. Các lệnh CLI thao tác đẩy Image (Push Commands)

Mở Terminal / Command Prompt tại máy phát triển và chạy chuỗi lệnh sau (Thay `<ACCOUNT_ID>` bằng ID tài khoản AWS của bạn):

#### **Bước A: Đăng nhập vào Amazon ECR Registry**

Sử dụng AWS CLI để lấy Token xác thực và đăng nhập Docker vào ECR:

```bash
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-northeast-1.amazonaws.com

```

*(Nếu hiển thị `Login Succeeded` tức là đăng nhập thành công).*

#### **Bước B: Biên dịch Docker Image cục bộ**

Biên dịch ứng dụng Spring Boot thành Image mang tên `enggo-backend`:

```bash
docker build -t enggo-backend .

```

#### **Bước C: Gán Tag (Đánh nhãn) Image theo chuẩn ECR**

Gán thẻ `latest` (hoặc theo số Version release) kèm theo đường dẫn ECR URI:

```bash
docker tag enggo-backend:latest <ACCOUNT_ID>[.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest](https://.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest)

```

#### **Bước D: Đẩy Image lên Amazon ECR**

Thực hiện lệnh đẩy Image đã gắn tag lên đám mây AWS ECR:

```bash
docker push <ACCOUNT_ID>[.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest](https://.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest)

```


*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình tab Images trong enggo-backend repository trên AWS Console hiển thị tag latest và kết quả Image scan.*

---

## 4. TỔNG KẾT VÀ QUY CHUẨN ĐÓNG GÓI

| Thông số / Tiêu chuẩn | Giá trị thiết lập | Mục đích & Lợi ích kỹ thuật |
| --- | --- | --- |
| **Repository Name** | `enggo-backend` | Định danh kho chứa bản đóng gói cho Spring Boot API |
| **Visibility** | `Private` | Đảm bảo mã nguồn và file thực thi không bị rò rỉ ra ngoài |
| **AWS Region** | `ap-northeast-1` (Tokyo) | Đồng bộ vùng với VPC, RDS và S3 giúp rút ngắn thời gian Deploy |
| **Image Scan Mode** | `Scan on push` | Phát hiện sớm các lỗ hổng bảo mật CVEs trong hệ điều hành và thư viện |
| **Tagging Strategy** | `:latest` & `:<commit-hash>` | Quản lý phiên bản phần mềm minh bạch, dễ dàng Rollback khi xảy ra lỗi |

```

```