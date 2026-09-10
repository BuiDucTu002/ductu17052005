---
title: "Khởi tạo EC2 và Redis"
weight: 4
chapter: false
pre: "<b>5.4.4. </b>"
date: 2026-09-10
---

# Khởi tạo EC2 và Redis

Dưới đây là nội dung báo cáo chi tiết cho phần **Triển khai ứng dụng Spring Boot trực tiếp trên máy chủ Amazon EC2**, được biên soạn chuẩn theo định dạng **Markdown**. Phần này hoàn thiện chuỗi báo cáo hạ tầng của bạn, bổ sung giải pháp thay thế đơn giản, tối ưu chi phí và dễ quản trị trực tiếp so với ECS Fargate.

Bạn chỉ cần sao chép (copy) đoạn Markdown dưới đây dán tiếp vào file tài liệu báo cáo `.md` của mình.

---

# TRIỂN KHAI ỨNG DỤNG SPRING BOOT BACKEND TRÊN AMAZON EC2

---

## 1. TỔNG QUAN GIẢI PHÁP TRIỂN KHAI TRÊN AMAZON EC2

Bên cạnh giải pháp triển khai container trên Amazon ECS, việc triển khai ứng dụng Backend `enggo-backend` trực tiếp trên **Amazon EC2 (Elastic Compute Cloud)** là một phương án hạ tầng mang lại nhiều ưu điểm thực tế:

* **Tối ưu hóa Chi phí (Cost Optimization):** Sử dụng các dòng Instance dung lượng nhỏ như `t3.micro` / `t3.small` (nằm trong gói AWS Free Tier hoặc chi phí cực thấp) giúp tối ưu ngân sách vận hành.
* **Dễ dàng Quản trị & Trực quan (Operational Simplicity):** Cho phép truy cập trực tiếp qua SSH để kiểm tra hệ thống, xem log trực tiếp (`tail -f`) và linh hoạt tinh chỉnh cấu hình runtime.
* **An toàn & Bảo mật cao:** Máy chủ EC2 được đặt trong VPC (`enggo-vpc`), kết nối đến CSDL Amazon RDS hoàn toàn thông qua dải IP nội bộ mà không cần mở kết nối Public (`Publicly Accessible = No`).

![Mô hình triển khai Spring Boot trên Amazon EC2 kết nối Amazon RDS](IMAGES/ec2-deployment-architecture.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh sơ đồ vị trí EC2 Instance gán enggo-ec2-sg, nằm trong Public/Private Subnet kết nối nội bộ tới RDS MySQL qua Port 3306.*

---

## 2. QUY TRÌNH THỰC HIỆN CHI TIẾT

### Bước 1: Khởi tạo và Cấu hình Amazon EC2 Instance

1. Đăng nhập vào **AWS Management Console** $\rightarrow$ Chọn dịch vụ **EC2** $\rightarrow$ Nhấn **Launch Instance**.
2. **Định danh & Hệ điều hành:**
   * **Name:** Nhập `enggo-backend-server`.
   * **Application and OS Images (AMI):** Chọn `Ubuntu Server 22.04 LTS (HVM)` hoặc `Amazon Linux 2023`.
3. **Cấu hình Phần cứng (Instance Type & Key Pair):**
   * **Instance type:** Chọn `t3.micro` hoặc `t3.small` (2 vCPU, 1-2 GB RAM).
   * **Key pair (login):** Chọn Key Pair có sẵn hoặc tạo mới (`enggo-key.pem`) để phục vụ kết nối SSH.
4. **Cấu hình Mạng & Tường lửa (`enggo-ec2-sg`):**
   * **Network:** Chọn `enggo-vpc`.
   * **Subnet:** Chọn `enggo-public-subnet-1` (Hoặc Private Subnet nếu có Bastion/NAT).
   * **Auto-assign public IP:** Chọn `Enable` (Để có IP kết nối SSH và gọi API trực tiếp).
   * **Create Security Group:** Đặt tên `enggo-ec2-sg`.
   * **Inbound Security Group Rules:**
     * *Rule 1 (Quản trị):* Type `SSH` | Port `22` | Source `My IP` (Chỉ mở quyền truy cập SSH cho IP máy cá nhân).
     * *Rule 2 (Ứng dụng):* Type `Custom TCP` | Port `8080` | Source `Anywhere-IPv4 (0.0.0.0/0)` (Mở cổng API cho Frontend/Mobile gọi tới).
5. Nhấn **Launch Instance**.

![Khởi tạo EC2 Instance trên AWS Console](IMAGES/step1-launch-ec2-instance.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình trang Launch Instance với thông số Ubuntu 22.04, t3.micro và Security Group enggo-ec2-sg.*

---

### Bước 2: Cập nhật Security Group của Cơ sở dữ liệu RDS (`enggo-rds-sg`)

Để đảm bảo nguyên tắc bảo mật tối thiểu (Principle of Least Privilege), cần ủy quyền cho nhóm `enggo-ec2-sg` mới khởi tạo được phép kết nối vào MySQL RDS qua cổng 3306.

1. Truy cập dịch vụ **RDS** $\rightarrow$ Chọn **Databases** $\rightarrow$ Chọn `enggo-db-instance`.
2. Truy cập tab **Connectivity & security** $\rightarrow$ Nhấp vào liên kết của **VPC security groups** (`enggo-rds-sg`).
3. Chọn tab **Inbound rules** $\rightarrow$ Nhấn **Edit inbound rules**.
4. Cập nhật / Thêm quy tắc mới:
   * **Type:** `MYSQL/Aurora` (Port `3306`).
   * **Source:** Chọn `Custom` $\rightarrow$ Nhập và chọn ID của **`enggo-ec2-sg`**.
5. Nhấn **Save rules**.

![Ủy quyền Security Group EC2 vào RDS Security Group](IMAGES/step2-rds-update-sg.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình trang Edit Inbound Rules của enggo-rds-sg chứa Source trỏ tới enggo-ec2-sg.*

---

### Bước 3: Cài đặt Môi trường Runtime & Triển khai Ứng dụng

Thực hiện kết nối SSH từ máy tính cá nhân vào máy chủ EC2 thông qua Terminal:

```bash
ssh -i "enggo-key.pem" ubuntu@<PUBLIC_IP_CUA_EC2>

```

Phương án chuẩn hóa môi trường thực thi, giúp cách ly dependencies và dễ dàng tích hợp với Amazon ECR.

##### 1. Cài đặt và khởi chạy Docker Engine:

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
# Cấp quyền chạy Docker không cần sudo (Tùy chọn)
sudo usermod -aG docker ubuntu

```

##### 2. Kéo (Pull) Docker Image từ Amazon ECR và Khởi chạy Container:

```bash
# Đăng nhập ECR
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-northeast-1.amazonaws.com

# Khởi chạy Container ngầm (Detached mode)
docker run -d \
  --name enggo-backend-app \
  -p 8080:8080 \
  -e SPRING_DATASOURCE_URL="jdbc:mysql://[enggo-db-instance.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com:3306/enggo_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC](https://enggo-db-instance.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com:3306/enggo_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC)" \
  -e SPRING_DATASOURCE_USERNAME="admin" \
  -e SPRING_DATASOURCE_PASSWORD="EngGoPass123!" \
  --restart always \
  <ACCOUNT_ID>[.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest](https://.dkr.ecr.ap-northeast-1.amazonaws.com/enggo-backend:latest)

```


*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình kết quả lệnh docker ps hiển thị container enggo-backend-app trạng thái Up trên port 8080.*

---

## 3. TỔNG KẾT VÀ ĐÁNH GIÁ MÔ HÌNH TRIỂN KHAI

| Tiêu chí | Cấu hình / Giải pháp | Hiệu quả kỹ thuật |
| --- | --- | --- |
| **Compute Node** | EC2 `t3.micro` / Ubuntu 22.04 LTS | Dễ dàng quản trị, tối ưu chi phí hạ tầng trong giai đoạn phát triển |
| **Network Security** | `enggo-ec2-sg` (Mở Port 22 SSH & 8080 API) | Giới hạn quyền SSH theo IP quản trị, mở cổng 8080 tiếp nhận request |
| **Database Security** | Kết nối nội bộ qua SG Referencing | CSDL RDS không mở Public Access, chỉ chấp nhận kết nối từ `enggo-ec2-sg` |
| **Deployment Standard** | Docker Container via Amazon ECR | Đóng gói đồng nhất, tự động khởi động lại (`--restart always`) khi OS reboot |

