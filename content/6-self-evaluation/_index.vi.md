---
title: "Tự đánh giá"
pre: "<b>6. </b>"
date: 2026-09-10
weight: 6
chapter: false
---

# 6. TỰ ĐÁNH GIÁ

Trải qua quá trình nghiên cứu, thiết kế và triển khai thực tế hệ thống hạ tầng đám mây cho ứng dụng **EngGo** trên Amazon Web Services (AWS), sinh viên/nhóm phát triển đưa ra các đánh giá tổng quan về mức độ hoàn thành nhiệm vụ như sau:

---

### 6.1. Kết quả đạt được (Successes)

* **Thiết lập Hạ tầng Mạng An toàn (Network Architecture):**
  * Xây dựng thành công VPC (`enggo-vpc`) với kiến trúc phân tầng Private/Public Subnet đa vùng khả dụng (Multi-AZ), đảm bảo tính sẵn sàng cao.
  * Thiết lập cơ chế kiểm soát truy cập phân tầng (Security Group Tiering), chỉ cho phép luồng dữ liệu hợp lệ đi qua và ngăn chặn hoàn toàn các truy cập trái phép.

* **Triển khai Dịch vụ Cơ sở Dữ liệu & Lưu trữ (Database & Storage):**
  * Khởi tạo thành công CSDL **Amazon RDS MySQL** (`enggo-db-instance`) nằm ẩn hoàn toàn trong Private Subnet (`Publicly Accessible = No`), đảm bảo an toàn dữ liệu tuyệt đối.
  * Tích hợp thành công **Amazon S3** (`enggo-media-bucket-2026`) lưu trữ tài nguyên truyền thông (ảnh đại diện, audio bài học), cấu hình chuẩn Bucket Policy và CORS cho phép ứng dụng truy xuất nhanh chóng.

* **Đóng gói Container & Tự động hóa CI/CD:**
  * Đóng gói ứng dụng Spring Boot Backend thành Docker Image tối ưu (Multi-stage build) và quản lý tập trung trên **Amazon ECR** (`enggo-backend`).
  * Xây dựng thành công pipeline CI/CD tự động bằng **GitHub Actions**: Tự động Build, Push Image lên ECR và Deploy lên **Amazon EC2** mỗi khi đẩy code mới, rút ngắn thời gian triển khai từ 20 phút xuống dưới 3 phút.

---

### 6.2. Hạn chế và Hướng phát triển (Limitations & Future Improvements)

| Thành phần | Hạn chế hiện tại | Hướng nâng cấp & Phát triển trong tương lai |
| :--- | :--- | :--- |
| **Tính sẵn sàng (High Availability)** | Ứng dụng chạy trên 1 EC2 Instance duy nhất, tiềm ẩn nguy cơ Single Point of Failure (SPOF) | Triển khai **Auto Scaling Group (ASG)** kết hợp **Application Load Balancer (ALB)** để tự động mở rộng theo lượng Traffic |
| **Bảo mật Credential** | Biến môi trường CSDL vẫn đang truyền trực tiếp vào lệnh chạy Container | Tích hợp **AWS Secrets Manager** hoặc **Systems Manager Parameter Store** để quản lý mật khẩu an toàn hơn |
| **Giám sát (Monitoring)** | Mới dừng lại ở việc xem log trực tiếp bằng lệnh `docker logs` | Tích hợp **Amazon CloudWatch Logs** và thiết lập Cảnh báo (CloudWatch Alarms) khi CPU/RAM vượt ngưỡng |
| **Bộ nhớ đệm (Caching)** | Chưa triển khai caching cho bài học và session | Tích hợp **Amazon ElastiCache (Redis)** để tối ưu tốc độ phản hồi API và xử lý Matchmaking PvP real-time |

---