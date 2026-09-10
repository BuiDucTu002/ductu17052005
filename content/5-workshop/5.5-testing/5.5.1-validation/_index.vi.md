---
title: "Kiểm thử kết quả"
weight: 1
chapter: false
pre: "<b>5.5.1. </b>"
date: 2026-09-10
---

### 5.5.1. Kiểm thử kết quả

Sau khi hoàn tất quá trình thiết lập hạ tầng mạng, khởi tạo cơ sở dữ liệu Amazon RDS, dịch vụ lưu trữ Amazon S3, ECR và triển khai ứng dụng Spring Boot lên Amazon EC2, tiến hành kiểm thử toàn diện các thành phần để xác minh tính sẵn sàng của hệ thống.

---

#### 1. Kiểm thử kết nối mạng và dịch vụ Backend (EC2 API)

Kiểm tra khả năng tiếp nhận yêu cầu HTTP từ môi trường Internet tới ứng dụng Spring Boot thông qua địa chỉ Public IP của EC2.

* **Thao tác:** Sử dụng lệnh `curl` hoặc công cụ Postman gửi yêu cầu tới Endpoint `/actuator/health` (hoặc API Ping/Health Check của hệ thống).
* **Lệnh thực thi:**
  ```
  curl -I http://<EC2_PUBLIC_IP>:8080/actuator/health```


* **Kết quả kỳ vọng:** Máy chủ trả về HTTP Status Code `200 OK` cùng phản hồi JSON biểu thị trạng thái `{"status":"UP"}`.


*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình Postman hoặc Terminal gửi request tới EC2 Public IP nhận kết quả HTTP 200 OK.*

---

#### 2. Kiểm thử kết nối Cơ sở dữ liệu nội bộ (EC2 to RDS MySQL)

Kiểm tra khả năng giao tiếp giữa container Spring Boot và Amazon RDS MySQL qua dải mạng nội bộ VPC (Port 3306).

* **Thao tác:**
1. SSH trực tiếp vào EC2 Instance (`enggo-backend-server`).
2. Thực hiện kết nối tới RDS Endpoint bằng MySQL Client hoặc kiểm tra log khởi tạo Hibernate/JPA của ứng dụng.


* **Lệnh thực thi (trên EC2):**
```bash
# Kiểm tra kết nối cổng mạng
nc -zv enggo-db-instance.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com 3306

# Kiểm tra log ứng dụng Spring Boot
docker logs -f enggo-backend-app

```


* **Kết quả kỳ vọng:**
* Lệnh `nc` trả về: `Connection to enggo-db-instance... port 3306 [tcp/mysql] succeeded!`.
* Log ứng dụng hiển thị thông báo kết nốiHikariPool thành công và tự động tạo/cập nhật cấu trúc bảng (`HikariPool-1 - Start completed.`, `Table "users" created`).




*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình Terminal hiển thị kết quả nc 3306 thành công và log Docker khởi tạo Hibernate schema.*

---

#### 3. Kiểm thử tải tệp Media lên Amazon S3 (Upload Avatar/Audio)

Kiểm tra luồng tải tệp tin từ client qua Backend API, đẩy trực tiếp lên Amazon S3 Bucket và truy xuất tệp qua URL công khai.

* **Thao tác:** Gửi request `POST` dạng `multipart/form-data` chứa file ảnh/audio tới endpoint `/api/v1/media/upload`.
* **Kết quả kỳ vọng:**
* API trả về HTTP Status `200 OK` chứa đường dẫn dạng: `https://enggo-media-bucket-2026.s3.ap-northeast-1.amazonaws.com/<UUID>_avatar.png`.
* Truy cập trực tiếp đường dẫn trên bằng trình duyệt, ảnh/file hiển thị thành công (Xác nhận Bucket Policy `PublicReadGetObject` hoạt động đúng).




*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình gửi request upload file qua Postman và ảnh mở tệp trên trình duyệt từ URL S3.*

---

#### 4. Bảng tổng hợp kết quả kiểm thử hệ thống

| Thành phần kiểm thử | Phương pháp thực hiện | Trạng thái | Ghi chú |
| --- | --- | --- | --- |
| **Hạ tầng VPC & IGW** | Routing Traffic từ Internet vào Public Subnet | **PASSED** | Kết nối mạng thông suốt |
| **Spring Boot API (EC2)** | Yêu cầu HTTP GET/POST tới Port 8080 | **PASSED** | Phản hồi < 100ms |
| **RDS MySQL (Private)** | Kết nối nội bộ Port 3306 từ `enggo-app-sg` | **PASSED** | Chặn hoàn toàn kết nối từ ngoài Internet |
| **Amazon S3 Integration** | Gọi `S3Service.uploadFile()` và fetch URL | **PASSED** | CORS & Policy đọc công khai hoạt động đúng |



