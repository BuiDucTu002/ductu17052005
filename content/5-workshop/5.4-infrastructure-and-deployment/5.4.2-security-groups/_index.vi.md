---
title: "Thiết lập Security Groups"
weight: 2
chapter: false
pre: "<b>5.4.2. </b>"
date: 2026-09-10
---

# Thiết lập Security Groups
### Bước 5: Thiết lập Security Groups theo kiến trúc Đa tầng (Security Group Tiering)

Security Group đóng vai trò là một tường lửa ảo (Stateful Virtual Firewall) ở cấp độ Instance/Container, kiểm soát trực tiếp các luồng dữ liệu vào (Inbound) và ra (Outbound). Hệ thống thiết lập quy tắc phân tầng (**Tiering Security**) nhằm triệt tiêu hoàn toàn khả năng truy cập trái phép vào Cơ sở dữ liệu.

![Sơ đồ nguyên lý hoạt động của Security Group Tiering](IMAGES/step5-sg-architecture.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh sơ đồ thể hiện luồng: Internet -> [enggo-app-sg] (Port 80/8080) -> [enggo-rds-sg] (Port 3306).*

#### Các bước thực hiện:

1. Đăng nhập vào **VPC Console** $\rightarrow$ Chọn menu **Security groups** $\rightarrow$ Nhấn **Create security group**.
2. **Khởi tạo Security Group cho Spring Boot / Load Balancer (`enggo-app-sg`):**
   * **Security group name:** `enggo-app-sg`.
   * **Description:** `Allow HTTP and API traffic to App/ALB`.
   * **VPC:** Chọn `enggo-vpc`.
   * **Inbound rules (Quy tắc đầu vào):**
     * *Rule 1:* Type `HTTP` | Port `80` | Source `Anywhere-IPv4 (0.0.0.0/0)` (Mở cổng truy cập Web tiêu chuẩn).
     * *Rule 2:* Type `Custom TCP` | Port `8080` | Source `Anywhere-IPv4 (0.0.0.0/0)` (Phục vụ kiểm thử và gọi API/WebSocket trực tiếp).
   * Nhấn **Create security group**.

![Cấu hình Inbound Rules cho enggo-app-sg](IMAGES/step5-app-sg-rules.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình trang cấu hình Inbound Rules của enggo-app-sg đã thêm Port 80 và 8080.*

3. **Khởi tạo Security Group cho MySQL RDS (`enggo-rds-sg`):**
   * **Security group name:** `enggo-rds-sg`.
   * **Description:** `Allow MySQL access ONLY from enggo-app-sg`.
   * **VPC:** Chọn `enggo-vpc`.
   * **Inbound rules (Quy tắc đầu vào):**
     * Type `MYSQL/Aurora` | Port `3306` | Source Chọn `Custom` $\rightarrow$ Nhập và chọn đúng ID của **`enggo-app-sg`** vừa tạo ở trên.
   * Nhấn **Create security group**.

![Cấu hình Inbound Rules cho enggo-rds-sg trỏ Source về enggo-app-sg](IMAGES/step5-rds-sg-rules.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình cấu hình Inbound Rule của enggo-rds-sg với Source chính là ID của enggo-app-sg (dạng sg-xxxxxxxx).*

#### Bảng tổng hợp cấu hình Security Groups:

| Tên Security Group | Cổng (Port) | Giao thức | Nguồn truy cập (Source) | Mục đích bảo mật |
| :--- | :--- | :--- | :--- | :--- |
| **`enggo-app-sg`** | `80`, `8080` | TCP | `0.0.0.0/0` | Tiếp nhận luồng Traffic REST API / WebSocket từ người dùng |
| **`enggo-rds-sg`** | `3306` | TCP | **`sg-xxxxxxxx` (`enggo-app-sg`)** | **CHỈ CHO PHÉP** ứng dụng Spring Boot kết nối vào CSDL |

---

## 3. TỔNG KẾT HẠ TẦNG VÀ ĐÁNH GIÁ MỤC TIÊU BẢO MẬT

Sau khi hoàn tất 5 bước khởi tạo, hệ thống đã sở hữu một **hạ tầng mạng chuẩn mực, hoàn chỉnh và đạt độ bảo mật cao** trên cloud AWS, sẵn sàng cho việc triển khai ứng dụng thực tế:

* **Phân vùng CSDL An toàn (Database Security):** Cơ sở dữ liệu **Amazon RDS MySQL** được triển khai nằm sâu trong `enggo-private-subnet` và gán nhóm `enggo-rds-sg`. Cấu hình này giúp ngăn chặn hoàn toàn các truy cập từ bên ngoài Internet (chống tấn công dò quét IP, Brute Force mật khẩu DB).
* **Kiến trúc Bảo mật Phân tầng (Tiered Security Model):** Nhờ cơ chế liên kết Security Group (SG-to-SG Reference), CSDL MySQL chỉ chấp nhận các kết nối mạng khởi tạo từ các máy chủ/container chứa ứng dụng **Spring Boot** mang nhóm `enggo-app-sg`.
* **Sẵn sàng triển khai ứng dụng (Deployment Ready):** 
  * Ứng dụng **Spring Boot (ECS Task / EC2 Instance)** có thể chạy linh hoạt tại Public Subnet hoặc Private Subnet với sự hỗ trợ của `enggo-app-sg`.
  * Hạ tầng sẵn sàng tích hợp các dịch vụ nâng cao như **AWS ECR** (quản lý Docker Image), **Redis Container** (xử lý Matchmaking PvP & WebSocket Session) và **Amazon S3** (lưu trữ Audio/Media bài học).