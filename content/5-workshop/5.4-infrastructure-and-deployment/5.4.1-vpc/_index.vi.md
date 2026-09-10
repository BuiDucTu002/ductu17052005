---
title: "Khởi tạo VPC và Subnets"
weight: 1
chapter: false
pre: "<b>5.4.1. </b>"
date: 2026-09-10
---

# Khởi tạo VPC và Subnets

# BÁO CÁO KỸ THUẬT: THIẾT KẾ VÀ TRIỂN KHAI HẠ TẦNG MẠNG VPC TRÊN AWS

---

## 1. TỔNG QUAN HẠ TẦNG MẠNG

Để đảm bảo tính sẵn sàng cao (**High Availability - HA**), khả năng mở rộng linh hoạt và tuân thủ chặt chẽ các quy chuẩn bảo mật đám mây (**AWS Well-Architected Framework**), hệ thống hạ tầng mạng riêng **Amazon VPC (Virtual Private Cloud)** cho ứng dụng **EngGo** được thiết kế theo kiến trúc phân tầng chuyên biệt.

Kiến trúc này giúp cách ly hoàn toàn môi trường tính toán ứng dụng (`Spring Boot`) và cơ sở dữ liệu (`RDS MySQL`) khỏi các nguy cơ tấn công trực tiếp từ Internet, đồng thời tối ưu hóa khả năng dự phòng thảm họa trên hai vùng khả dụng (**Availability Zones - AZs**).

![Sơ đồ kiến trúc hạ tầng mạng VPC EngGo](IMAGES/vpc-architecture-diagram.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Thay thế đường dẫn trên bằng hình ảnh Sơ đồ kiến trúc VPC tổng quan (mô tả VPC, 2 AZs, Public/Private Subnets, IGW, Route Tables).*

---

### 1.1. Các thành phần cốt lõi của hạ tầng
* **01 Amazon VPC:** Không gian mạng ảo độc lập với dải địa chỉ IP nội bộ `10.0.0.0/16` (cung cấp $65,536$ địa chỉ IP khả dụng).
* **01 Internet Gateway (IGW):** Cổng kết nối trung gian chuyển tiếp lưu lượng giữa mạng nội bộ VPC và Internet.
* **02 Public Subnets:** Phân vùng mạng công khai đặt tại 2 AZs khác nhau (`ap-northeast-1a` và `ap-northeast-1c`), dùng để triển khai **Application Load Balancer (ALB)** hoặc máy chủ giao tiếp công khai.
* **02 Private Subnets:** Phân vùng mạng nội bộ được bảo vệ tuyệt đối, không có tuyến đường trực tiếp ra Internet, dùng để triển khai ứng dụng **Spring Boot Container (ECS/EC2)** và cơ sở dữ liệu **Amazon RDS MySQL**.
* **02 Route Tables:** Quản lý và điều hướng luồng dữ liệu cho từng phân vùng Subnet (Public Route Table & Private Route Table).

---

## 2. QUY TRÌNH TRIỂN KHAI CHI TIẾT

### Bước 1: Khởi tạo Amazon VPC (`enggo-vpc`)

Khởi tạo mạng riêng ảo đóng vai trò là "bức tường bao" bảo vệ toàn bộ tài nguyên của ứng dụng EngGo trên đám mây AWS (Region Tokyo: `ap-northeast-1`).

#### Các bước thực hiện:
1. Đăng nhập vào **AWS Management Console**, tìm kiếm và truy cập dịch vụ **VPC**.
2. Tại thanh điều hướng bên trái, chọn **Your VPCs** $\rightarrow$ Nhấn **Create VPC**.
3. Cấu hình các thông số kỹ thuật:
   * **Resources to create:** Chọn `VPC only`.
   * **Name tag:** Nhập `enggo-vpc`.
   * **IPv4 CIDR block:** Chọn `IPv4 CIDR manual input`.
   * **IPv4 CIDR:** Nhập `10.0.0.0/16`.
   * **Tenancy:** Chọn `Default`.
4. Nhấn **Create VPC** để hoàn tất.

![Giao diện khởi tạo VPC trên AWS Console](IMAGES/step1-create-vpc.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình trang cấu hình "Create VPC" đã điền tên enggo-vpc và CIDR 10.0.0.0/16.*

#### Bảng tổng hợp thông số VPC:

| Thông số (Parameter) | Giá trị cấu hình (Value) | Mô tả kỹ thuật |
| :--- | :--- | :--- |
| **VPC Name** | `enggo-vpc` | Tên định danh tài nguyên hạ tầng mạng EngGo |
| **AWS Region** | `ap-northeast-1` (Tokyo) | Khu vực máy chủ vật lý triển khai tài nguyên |
| **IPv4 CIDR Block** | `10.0.0.0/16` | Cấp phát dải địa chỉ IP từ `10.0.0.0` đến `10.0.255.255` |
| **Tenancy** | `Default` | Hạ tầng phần cứng chia sẻ giúp tối ưu chi phí |

---

### Bước 2: Tạo và gắn Internet Gateway (`enggo-igw`)

Internet Gateway đóng vai trò là cửa ngõ duy nhất cho phép luồng dữ liệu từ Public Subnet giao tiếp với môi trường Internet bên ngoài.

#### Các bước thực hiện:
1. Tại thanh điều hướng bên trái, chọn **Internet gateways** $\rightarrow$ Nhấn **Create internet gateway**.
2. Tại mục **Name tag**, nhập `enggo-igw`.
3. Nhấn **Create internet gateway**.
4. Chọn cổng `enggo-igw` vừa tạo $\rightarrow$ Nhấn **Actions** $\rightarrow$ Chọn **Attach to VPC**.
5. Chọn VPC target là `enggo-vpc` và nhấn **Attach internet gateway**.

![Gán Internet Gateway vào VPC](IMAGES/step2-attach-igw.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình trạng thái enggo-igw đã Attached vào enggo-vpc thành công.*

---

### Bước 3: Phân chia Subnet công khai & nội bộ (Public/Private Subnets)

Để đạt tính sẵn sàng cao (HA), hệ thống chia 4 Subnets nằm trên 2 Availability Zones (`ap-northeast-1a` và `ap-northeast-1c`).

#### Bảng phân hoạch IP Subnet:

| Tên Subnet | Loại Subnet | Availability Zone | IPv4 CIDR Block | Mục đích sử dụng |
| :--- | :--- | :--- | :--- | :--- |
| `enggo-public-subnet-1` | Public | `ap-northeast-1a` | `10.0.1.0/24` | Load Balancer / Bastion Host |
| `enggo-public-subnet-2` | Public | `ap-northeast-1c` | `10.0.2.0/24` | Dự phòng cho Load Balancer |
| `enggo-private-subnet-1` | Private | `ap-northeast-1a` | `10.0.10.0/24` | Spring Boot App & RDS MySQL |
| `enggo-private-subnet-2` | Private | `ap-northeast-1c` | `10.0.20.0/24` | Dự phòng RDS Standby / ECS |

#### Các bước thực hiện:
1. Vào menu **Subnets** $\rightarrow$ Chọn **Create subnet**.
2. Chọn **VPC ID**: `enggo-vpc`.
3. Lần lượt khai báo 4 Subnets theo đúng thông số trong bảng trên.
4. Bật tính năng **Auto-assign public IPv4** cho 2 Public Subnet:
   * Chọn `enggo-public-subnet-1` $\rightarrow$ **Actions** $\rightarrow$ **Edit subnet settings**.
   * Tích chọn **Enable auto-assign public IPv4 addresses** $\rightarrow$ Nhấn **Save**.
   * Lặp lại thao tác tương tự cho `enggo-public-subnet-2`.

![Danh sách Subnets đã tạo trong VPC](IMAGES/step3-subnets-list.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình bảng danh sách 4 Subnets đã hoàn thành kèm dải IP CIDR tương ứng.*

---

### Bước 4: Cấu hình Bảng định tuyến (Route Tables)

Route Table quyết định luồng đi của gói tin giữa các Subnet và cấm hoàn toàn lưu lượng từ bên ngoài truy cập vào Private Subnet.

#### 1. Cấu hình Public Route Table (`enggo-public-rt`)
1. Vào menu **Route tables** $\rightarrow$ Nhấn **Create route table**.
2. Đặt tên: `enggo-public-rt`, chọn VPC: `enggo-vpc` $\rightarrow$ Nhấn **Create**.
3. Chọn tab **Routes** $\rightarrow$ Nhấn **Edit routes** $\rightarrow$ Thêm tuyến đường:
   * **Destination:** `0.0.0.0/0`
   * **Target:** `Internet Gateway` (Chọn `enggo-igw`).
4. Chọn tab **Subnet associations** $\rightarrow$ Nhấn **Edit subnet associations** $\rightarrow$ Tích chọn 2 Subnet: `enggo-public-subnet-1` và `enggo-public-subnet-2` $\rightarrow$ Nhấn **Save associations**.

![Cấu hình Route Table cho Public Subnet](IMAGES/step4-public-route-table.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình bảng Routes chứa rule 0.0.0.0/0 trỏ tới enggo-igw.*

#### 2. Cấu hình Private Route Table (`enggo-private-rt`)
1. Tạo Route Table mới tên: `enggo-private-rt`, chọn VPC: `enggo-vpc`.
2. Không thêm tuyến `0.0.0.0/0` trỏ ra IGW (Đảm bảo tính riêng tư hoàn toàn).
3. Tại tab **Subnet associations**, liên kết với 2 Subnet nội bộ: `enggo-private-subnet-1` và `enggo-private-subnet-2`.

---

## 3. KẾT QUẢ VÀ ĐÁNH GIÁ MỤC TIÊU BẢO MẬT

Sau khi hoàn thành việc triển khai, hạ tầng mạng `enggo-vpc` đạt được các tiêu chuẩn kỹ thuật đề ra:

1. **Phân tách môi trường an toàn:** Cơ sở dữ liệu RDS và các máy chủ ứng dụng Backend nằm hoàn toàn trong Private Subnet, ngăn chặn 100% các cuộc tấn công quét cổng (Port Scanning) hoặc DDoS từ môi trường Internet.
2. **Khả năng chịu lỗi (Fault Tolerance):** Hạ tầng được trải đều trên 2 vùng khả dụng (AZs: `1a` và `1c`), đảm bảo hệ thống duy trì hoạt động ngay cả khi một trung tâm dữ liệu AWS gặp sự cố.
3. **Sẵn sàng cho CI/CD & Auto Scaling:** Dải IP phong phú (`10.0.0.0/16`) cung cấp đủ không gian mạng cho việc mở rộng quy mô Container tự động và thiết lập luồng triển khai phần mềm CI/CD liên tục.