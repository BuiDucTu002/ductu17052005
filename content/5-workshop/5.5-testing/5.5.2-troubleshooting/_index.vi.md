---
title: "Troubleshooting"
weight: 2
chapter: false
pre: "<b>5.5.2. </b>"
date: 2026-09-10
---

### 5.5.2. Troubleshooting (Khắc phục sự cố thường gặp)

Trong quá trình khởi tạo hạ tầng AWS, đóng gói Container và triển khai ứng dụng Spring Boot, một số sự cố kỹ thuật có thể phát sinh do lỗi cấu hình mạng, phân quyền bảo mật hoặc thiếu hụt tài nguyên. Dưới đây là bảng tổng hợp các sự cố thường gặp và quy trình xử lý từng bước (Troubleshooting Guide):

---

#### 1. Sự cố 1: Spring Boot không thể kết nối tới Amazon RDS MySQL

* **Dấu hiệu:** Log ứng dụng báo lỗi `com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link failure` hoặc `HikariPool-1 - Exception during pool initialization`.
* **Nguyên nhân chính:**
  1. Security Group của RDS (`enggo-rds-sg`) chưa mở cổng `3306` cho Security Group của EC2 (`enggo-ec2-sg`).
  2. Đường dẫn Endpoint RDS hoặc thông tin tài khoản/mật khẩu trong file cấu hình không chính xác.
  3. RDS Instance và EC2 Instance nằm ở hai VPC khác nhau.
* **Quy trình khắc phục:**
  1. Kiểm tra Security Group `enggo-rds-sg`: Truy cập **RDS Console** $\rightarrow$ **Databases** $\rightarrow$ chọn `enggo-db-instance` $\rightarrow$ **Connectivity & security** $\rightarrow$ kiểm tra mục **Inbound rules**. Đảm bảo có rule cho cổng `3306` với Source trỏ tới đúng ID của `enggo-ec2-sg`.
  2. Kiểm tra kết nối mạng từ EC2: SSH vào máy chủ EC2 và kiểm tra xem cổng 3306 có thông không bằng lệnh:
     ```bash
     nc -zv <RDS_ENDPOINT> 3306
     ```
  3. Kiểm tra biến môi trường `SPRING_DATASOURCE_URL`, `USERNAME`, `PASSWORD` truyền vào Container đã khớp với thông số khởi tạo trên RDS hay chưa.

![Khắc phục lỗi kết nối RDS bằng cách kiểm tra Inbound Rules](IMAGES/troubleshoot-rds-sg.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình bảng cấu hình Inbound Rules của enggo-rds-sg mở Port 3306 cho enggo-ec2-sg.*

---

#### 2. Sự cố 2: Lỗi phân quyền 403 Forbidden khi Upload tệp lên Amazon S3

* **Dấu hiệu:** API trả về mã lỗi HTTP `403 Forbidden` hoặc log Java hiển thị `software.amazon.awssdk.services.s3.model.S3Exception: Access Denied`.
* **Nguyên nhân chính:**
  1. IAM User (`enggo-s3-user`) chưa được gán chính sách `AmazonS3FullAccess`.
  2. Bật tính năng *Block All Public Access* trên S3 Bucket nhưng chưa cấu hình Bucket Policy hoặc ACL phù hợp.
  3. Thông tin `AWS_ACCESS_KEY_ID` hoặc `AWS_SECRET_ACCESS_KEY` truyền vào Spring Boot bị sai hoặc hết hạn.
* **Quy trình khắc phục:**
  1. Kiểm tra IAM User: Truy cập **IAM Console** $\rightarrow$ **Users** $\rightarrow$ chọn `enggo-s3-user` $\rightarrow$ kiểm tra tab **Permissions** đảm bảo đã có policy `AmazonS3FullAccess`.
  2. Kiểm tra tab **Permissions** của S3 Bucket `enggo-media-bucket-2026`: Đảm bảo mục **Block public access** đã bỏ tích chọn cho phép đọc công khai và **Bucket policy** đã chứa đoạn mã `s3:GetObject` cho `Principal: "*"`.

---

#### 3. Sự cố 3: Lỗi kết nối SSH vào máy chủ EC2 (Connection Timed Out / Permission Denied)

* **Dấu hiệu:** Lệnh `ssh -i enggo-key.pem ubuntu@<PUBLIC_IP>` treo vô hạn hoặc báo `Permission denied (publickey)`.
* **Nguyên nhân chính:**
  1. Security Group `enggo-ec2-sg` chưa mở cổng `22` (SSH) cho IP hiện tại của người dùng.
  2. File khóa riêng tư `.pem` chưa được phân quyền an toàn (quyền quá rộng trên Linux/macOS).
  3. Nhập sai tên người dùng mặc định của OS (dùng `root` hoặc `ec2-user` thay vì `ubuntu`).
* **Quy trình khắc phục:**
  1. Cập nhật Inbound Rule cho `enggo-ec2-sg`: Mở cổng `22` với Source chọn `My IP`.
  2. Chỉnh sửa quyền file khóa SSH trên máy cá nhân trước khi kết nối:
     ```bash
     chmod 400 enggo-key.pem
     ```
  3. Xác nhận đúng Username hệ điều hành: Ubuntu (`ubuntu`), Amazon Linux (`ec2-user`).

![Chỉnh sửa quyền file PEM và SSH vào EC2 thành công](IMAGES/troubleshoot-ssh-ec2.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình lệnh chmod 400 và kết quả SSH vào Ubuntu EC2 thành công.*

---

#### 4. Sự cố 4: GitHub Actions thất bại ở bước "Deploy to EC2 via SSH"

* **Dấu hiệu:** Pipeline trên GitHub Actions báo lỗi tại bước SSH: `ssh: handshake failed: ssh: unable to authenticate` hoặc `dial tcp <IP>:22: i/o timeout`.
* **Nguyên nhân chính:**
  1. Giá trị Secret `EC2_SSH_KEY` dán vào GitHub bị thiếu dòng đầu/cuối (`-----BEGIN RSA PRIVATE KEY-----`) hoặc dán thừa khoảng trắng.
  2. Security Group của EC2 chặn IP động phát xuất từ các máy máy chủ runner của GitHub Actions.
* **Quy trình khắc phục:**
  1. Mở file `.pem` bằng Text Editor, copy toàn bộ nội dung (bao gồm cả dòng Header và Footer) và dán lại vào **GitHub Secrets** $\rightarrow$ `EC2_SSH_KEY`.
  2. Cấu hình Inbound Rule cổng `22` trên `enggo-ec2-sg` tạm thời cho phép `0.0.0.0/0` (hoặc sử dụng Action tự động lấy IP Runner thêm vào SG trước khi SSH).

---

#### 5. Bảng tóm tắt chẩn đoán nhanh lỗi hạ tầng

| Mã lỗi / Hiện tượng | Thành phần gây lỗi | Thao tác kiểm tra nhanh |
| :--- | :--- | :--- |
| **`Connection Timed Out` (Port 8080/22)** | Security Group / Route Table | Kiểm tra Inbound Rules của `enggo-ec2-sg` & Route Table trỏ ra IGW |
| **`Access Denied` (S3)** | IAM User / S3 Policy | Kiểm tra `AmazonS3FullAccess` và `Bucket Policy` |
| **`Communications Link Failure` (RDS)** | Security Group / VPC Subnet | Kiểm tra Inbound Rules cổng 3306 trên `enggo-rds-sg` |
| **`No space left on device` (EC2)** | Docker Images rác tích tụ | Thực thi lệnh `docker system prune -a -f` trên EC2 |