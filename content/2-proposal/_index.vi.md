---
title: "2. Đề xuất"
date: 2026-09-10
weight: 2
chapter: false
---

# Đề xuất

# Hệ thống Enggo-Backend Cloud-Native trên AWS

---

# 1. Tóm tắt

Enggo-Backend là một nền tảng ứng dụng backend hiện đại được xây dựng với Spring Boot và được thiết kế để triển khai trên các dịch vụ AWS được quản lý. Hệ thống cung cấp các API RESTful hoàn chỉnh, xác thực người dùng, quản lý dữ liệu, xử lý tệp, và các tính năng nâng cao khác thông qua nền tảng đám mây có khả năng mở rộng.

Ứng dụng được phát triển bằng **Java Spring Boot**, **Spring Data JPA**, **Spring Security** và sử dụng **PostgreSQL** hoặc **MySQL** làm cơ sở dữ liệu. Ứng dụng được đóng gói bằng **Docker** và triển khai trên **Amazon ECS Fargate** phía sau **Application Load Balancer (ALB)**. Docker Image được lưu trữ trong **Amazon ECR**, tệp dữ liệu được lưu trữ trong **Amazon S3**, và **AWS CodeBuild** tự động build và triển khai phiên bản mới nhất mỗi khi mã nguồn được đẩy lên GitHub.

Môi trường triển khai cũng sử dụng **Amazon RDS** để quản lý cơ sở dữ liệu, **Amazon Route 53** để quản lý tên miền, **AWS Certificate Manager (ACM)** để mã hóa HTTPS, **Amazon CloudWatch** để giám sát, **AWS IAM** để kiểm soát truy cập và **Amazon VPC** để bảo mật mạng. Kiến trúc này cung cấp triển khai tự động, lưu trữ tập trung, quản lý đơn giản và hạ tầng đám mây có khả năng mở rộng, phù hợp với các ứng dụng backend quy mô nhỏ đến vừa.

---

# 2. Vấn đề

## Vấn đề hiện tại

Nhiều ứng dụng backend truyền thống phải đối mặt với những thách thức sau:
- Quản lý cơ sở dữ liệu cục bộ phức tạp và khó mở rộng
- Triển khai thủ công yêu cầu nhiều công sức vận hành
- Khó khăn trong việc quản lý tài nguyên và chi phí cơ sở hạ tầng
- Thời gian ngừng hoạt động cao khi cập nhật hoặc sửa lỗi
- Khó bảo trì và mở rộng khi số lượng người dùng tăng

## Giải pháp

Giải pháp đề xuất là phát triển một nền tảng backend cloud-native sử dụng các dịch vụ AWS được quản lý.

Hệ thống Enggo-Backend cung cấp:
- **API RESTful** hoàn chỉnh cho các ứng dụng frontend
- **Xác thực an toàn** sử dụng Spring Security và JWT
- **Quản lý dữ liệu** tập trung qua Amazon RDS
- **Lưu trữ tệp** tin cậy qua Amazon S3
- **Triển khai tự động** thông qua CI/CD pipeline
- **Mở rộng tự động** bằng Amazon ECS Auto Scaling
- **Giám sát thực tế** qua Amazon CloudWatch
- **Bảo mật tối ưu** qua AWS IAM và VPC

## Lợi ích

Kiến trúc đề xuất mang lại các lợi ích sau:

- ✅ Đơn giản hóa triển khai ứng dụng backend
- ✅ Quy trình CI/CD tự động hoàn toàn
- ✅ Hạ tầng đám mây có khả năng mở rộng tự động
- ✅ Giao tiếp HTTPS/TLS an toàn
- ✅ Lưu trữ dữ liệu tin cậy với sao lưu tự động
- ✅ Đơn giản hóa bảo trì ứng dụng
- ✅ Giảm đáng kể công sức vận hành
- ✅ Dễ dàng mở rộng trong tương lai

---

# 3. Kiến trúc giải pháp

## Sơ đồ kiến trúc

Ứng dụng sử dụng kiến trúc cloud-native được triển khai trên các dịch vụ AWS được quản lý:

```
Client Applications
        ↓
Amazon Route 53 (DNS)
        ↓
AWS Certificate Manager (ACM - HTTPS)
        ↓
Application Load Balancer (ALB)
        ↓
Amazon ECS Fargate (Container Orchestration)
        ↓
┌───────────────────────────────────┐
│   Spring Boot Application          │
│   (Java Backend Service)           │
└───────────────────────────────────┘
        ↓         ↓         ↓
   Amazon    Amazon S3    Amazon
   RDS      (Storage)     ECR
   (DB)    (Image Files) (Registry)
   ↓
AWS CodeBuild & CodePipeline (CI/CD)
        ↓
GitHub Repository
```

## Các dịch vụ AWS sử dụng

| Dịch vụ | Chức năng |
|---------|----------|
| **Amazon VPC** | Mạng ảo riêng an toàn |
| **AWS IAM** | Quản lý quyền truy cập |
| **Amazon ECS Fargate** | Quản lý container không máy chủ |
| **Amazon ECR** | Kho lưu trữ Docker Image |
| **Amazon RDS** | Cơ sở dữ liệu có quản lý |
| **Amazon S3** | Lưu trữ dữ liệu từ xa |
| **Application Load Balancer (ALB)** | Cân bằng tải |
| **Amazon Route 53** | Quản lý tên miền DNS |
| **AWS Certificate Manager (ACM)** | Chứng chỉ HTTPS/TLS |
| **AWS CodeBuild** | Build Docker Image tự động |
| **AWS CodePipeline** | Quy trình CI/CD tự động |
| **Amazon CloudWatch** | Giám sát và logging |

## Thiết kế thành phần

### Backend

- **Ngôn ngữ:** Java
- **Framework:** Spring Boot, Spring Data JPA, Spring Security
- **API:** RESTful API, JSON
- **Xác thực:** JWT Token, Spring Security

### Cơ sở dữ liệu

- **Amazon RDS:** PostgreSQL hoặc MySQL
- **Sao lưu tự động:** Automated Backup
- **Multi-AZ:** Tính sẵn sàng cao

### Lưu trữ tệp

- **Amazon S3:** Lưu trữ tệp dữ liệu
- **CloudFront:** CDN tối ưu tốc độ (tùy chọn)

### Nền tảng Container

- **Docker:** Containerization
- **Amazon ECS Fargate:** Orchestration
- **Amazon ECR:** Docker Registry

### Quy trình triển khai

```
GitHub Repository (Push code)
            ↓
AWS CodeBuild (Build Docker Image)
            ↓
Amazon ECR (Store Docker Image)
            ↓
AWS CodeDeploy/ECS (Deploy to Fargate)
            ↓
Application Live
```

---

# 4. Triển khai kỹ thuật

## Các giai đoạn triển khai

Dự án được triển khai qua các giai đoạn sau:

1. **Tuần 1:** Nghiên cứu kiến trúc AWS, phân tích dự án Enggo-Backend
2. **Tuần 2:** Thiết kế kiến trúc, cấu hình VPC, chuẩn bị Amazon RDS
3. **Tuần 3:** Phát triển backend, cấu hình Docker, build Docker Image
4. **Tuần 4:** Triển khai Amazon ECS Fargate, ALB, Route 53, ACM
5. **Tuần 5:** Cấu hình CI/CD, giám sát CloudWatch, kiểm thử toàn hệ thống

## Yêu cầu kỹ thuật

### Ngôn ngữ & Framework

- Java
- Spring Boot
- Spring Data JPA
- Spring Security

### Cơ sở dữ liệu

- PostgreSQL / MySQL
- Amazon RDS

### Dịch vụ AWS

- Amazon VPC, AWS IAM
- Amazon ECS Fargate, Amazon ECR
- Amazon RDS, Amazon S3
- Application Load Balancer
- Amazon Route 53, AWS Certificate Manager
- AWS CodeBuild, AWS CodePipeline
- Amazon CloudWatch

### Công cụ phát triển

- IntelliJ IDEA / VS Code
- Git & GitHub
- Docker Desktop
- Maven / Gradle
- AWS CLI

---

# 5. Lộ trình & Các mốc

### Tuần 1 – Nghiên cứu & Phân tích

- Tìm hiểu về AWS cloud architecture và best practices
- Phân tích dự án Enggo-Backend
- Thiết kế kiến trúc tổng thể
- Chuẩn bị tài khoản AWS và IAM roles

### Tuần 2 – Thiết kế & Chuẩn bị Hạ tầng

- Thiết kế Amazon VPC và subnets
- Cấu hình Security Groups
- Tạo Amazon RDS database
- Chuẩn bị môi trường phát triển

### Tuần 3 – Phát triển & Containerization

- Phát triển/cập nhật Spring Boot backend
- Cấu hình Spring Data JPA
- Tạo Dockerfile
- Build Docker Image cục bộ

### Tuần 4 – Triển khai AWS

- Đẩy Docker Image lên Amazon ECR
- Tạo ECS Task Definition
- Triển khai lên Amazon ECS Fargate
- Cấu hình Application Load Balancer
- Cấu hình Amazon Route 53 & ACM

### Tuần 5 – CI/CD & Giám sát

- Cấu hình AWS CodeBuild
- Thiết lập CI/CD pipeline
- Cấu hình Amazon CloudWatch
- Kiểm thử toàn hệ thống

---

# 6. Ước tính chi phí

## Ước tính chi phí hạ tầng hàng tháng

| Dịch vụ | Chi phí ước tính |
|---------|-----------------|
| Amazon ECS Fargate | ~$5.00 |
| Amazon RDS (db.t3.micro) | ~$15.00 |
| Amazon S3 (Lưu trữ & Requests) | ~$1.00 |
| Amazon ECR | ~$0.50 |
| AWS CodeBuild | ~$1.00 |
| Application Load Balancer | ~$16.00 |
| Amazon CloudWatch (Logs & Metrics) | ~$2.00 |
| AWS Data Transfer | ~$5.00 |
| **Tổng ước tính** | **~$45.50/tháng** |

### Hướng dẫn kiểm soát chi phí

- **AWS Budgets:** Cảnh báo tự động khi chi phí vượt $50 và $100
- **Amazon RDS:** Sử dụng db.t3.micro cho development, hạn chế backup
- **ECS Fargate:** Tối ưu resource allocation cho CPU/Memory
- **AWS CodeBuild:** Chỉ build khi mã nguồn được đẩy lên GitHub
- **CloudWatch Logs:** Thiết lập retention policy để giảm chi phí
- **Dọn dẹp sau demo:** Xóa ECS services, RDS instances, ECR images, S3 buckets, ALB không sử dụng

---

# 7. Đánh giá rủi ro

## Ma trận rủi ro

| Rủi ro | Tác động | Xác suất |
|--------|----------|---------|
| Triển khai Amazon ECS thất bại | Cao | Trung bình |
| Lỗi kết nối Amazon RDS | Cao | Thấp |
| Tải lên Amazon ECR thất bại | Trung bình | Thấp |
| AWS CodeBuild build thất bại | Trung bình | Trung bình |
| Lỗi cấu hình DNS Route 53 | Trung bình | Thấp |
| Lỗi chứng chỉ HTTPS ACM | Thấp | Rất thấp |
| Chi phí vượt ngân sách | Cao | Trung bình |
| Hiệu suất application kém | Trung bình | Thấp |

## Chiến lược giảm thiểu

- ✅ Bật giám sát **Amazon CloudWatch** cho tất cả resources
- ✅ Cấu hình cảnh báo **AWS Budgets** định kỳ
- ✅ Quản lý phiên bản Docker Image bằng **Amazon ECR Lifecycle Policy**
- ✅ Bật **Multi-AZ** cho Amazon RDS
- ✅ Áp dụng chính sách **IAM theo nguyên tắc đặc quyền tối thiểu**
- ✅ Kiểm tra bản ghi **DNS Route 53** trước triển khai
- ✅ Kiểm tra trạng thái **chứng chỉ ACM** trước bật HTTPS
- ✅ Sao lưu **Amazon RDS** định kỳ

## Kế hoạch dự phòng

- Khôi phục **Docker Image** trước đó từ Amazon ECR
- Triển khai lại **ECS Task Definition** trước đó
- Khôi phục **bản sao lưu Amazon RDS**
- Triển khai lại qua **AWS CodeBuild**
- Cấu hình lại **bản ghi DNS Route 53**
- Cấp lại **chứng chỉ ACM** khi xác thực thất bại

---

# 8. Kết quả mong đợi

## Kết quả kỹ thuật

Dự án hoàn thành sẽ cung cấp:

✅ **Hệ thống backend cloud-native** hoàn chỉnh  
✅ **Spring Boot application** được container hóa  
✅ **Cơ sở dữ liệu có quản lý** trên Amazon RDS  
✅ **Triển khai CI/CD tự động** bằng GitHub & AWS CodeBuild  
✅ **Container orchestration** bằng Amazon ECS Fargate  
✅ **Cân bằng tải** bằng Application Load Balancer  
✅ **Giao tiếp HTTPS/TLS an toàn** bằng AWS Certificate Manager  
✅ **Quản lý tên miền** bằng Amazon Route 53  
✅ **Lưu trữ tệp tin cậy** bằng Amazon S3  
✅ **Giám sát toàn diện** bằng Amazon CloudWatch  
✅ **Quản lý truy cập an toàn** bằng AWS IAM  

## Giá trị kinh doanh

Dự án minh họa việc triển khai thực tế ứng dụng backend hiện đại sử dụng Spring Boot trên các dịch vụ AWS được quản lý, container hóa và DevOps.

Kiến trúc cloud-native giúp:
- Đơn giản hóa triển khai backend
- Giảm công sức vận hành
- Cải thiện khả năng mở rộng
- Giảm thời gian phát triển tính năng mới
- Cung cấp nền tảng tin cậy cho mở rộng

Các cải tiến trong tương lai có thể bao gồm:
- Tích hợp **AWS Lambda** cho serverless functions
- Sử dụng **Amazon DynamoDB** cho NoSQL database
- Tích hợp **Amazon SQS/SNS** cho message queue
- Triển khai **microservices architecture**
- Thêm **caching layer** với Amazon ElastiCache
- Tích hợp **authentication** với AWS Cognito

