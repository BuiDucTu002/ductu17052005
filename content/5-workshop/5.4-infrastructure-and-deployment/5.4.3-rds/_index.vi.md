---
title: "Khởi tạo RDS MySQL"
weight: 3
chapter: false
pre: "<b>5.4.3. </b>"
date: 2026-09-10
---

# Khởi tạo RDS MySQL vs S3

---

# TRIỂN KHAI CƠ SỞ DỮ LIỆU AMAZON RDS MYSQL TRONG PRIVATE SUBNET

---

## 1. TỔNG QUAN HẠ TẦNG CƠ SỞ DỮ LIỆU

Cơ sở dữ liệu (**Database Layer**) là thành phần cốt lõi lưu trữ toàn bộ dữ liệu người dùng, tiến trình học tập, cấu hình bài thi và lịch sử đấu PvP của ứng dụng **EngGo**. 

Để đạt được tiêu chuẩn bảo mật đám mây cao nhất, dịch vụ cơ sở dữ liệu quản trị **Amazon RDS (Relational Database Service) MySQL** được triển khai nằm ẩn sâu bên trong các **Private Subnets** thuộc hạ tầng `enggo-vpc`.

![Sơ đồ vị trí Amazon RDS trong hạ tầng VPC EngGo](IMAGES/rds-architecture-diagram.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh sơ đồ vị trí RDS đặt trong Private Subnet, không có kết nối trực tiếp ra Internet và chỉ nhận traffic từ enggo-app-sg qua cổng 3306.*

### Chiêu thức đảm bảo Bảo mật & Tối ưu:
* **Cách ly hoàn toàn (Network Isolation):** Cấu hình `Public Access = No` kết hợp việc đặt trong Private Subnet giúp vô hiệu hóa mọi đường kết nối từ ngoài Internet.
* **Tường lửa phân tầng (Security Group Tiering):** Gán nhóm `enggo-rds-sg`, chỉ cho phép các truy cập TCP trên cổng `3306` xuất phát từ ứng dụng Backend Spring Boot (`enggo-app-sg`).
* **Sẵn sàng cho Dự phòng (Multi-AZ Ready):** Gom các Private Subnets ở các Availability Zones khác nhau vào một **DB Subnet Group** giúp RDS sẵn sàng mở rộng hoặc chuyển vùng dự phòng khi có sự cố.

---

## 2. QUY TRÌNH THỰC HIỆN CHI TIẾT

### Bước 1: Khởi tạo DB Subnet Group (`enggo-db-subnet-group`)

DB Subnet Group là tập hợp các Subnet nội bộ (Private Subnets) mà Amazon RDS được phép sử dụng để cấp phát địa chỉ IP và tài nguyên máy chủ CSDL.

#### Các bước thực hiện:
1. Đăng nhập vào **AWS Management Console** $\rightarrow$ Tìm kiếm và chọn dịch vụ **RDS**.
2. Tại thanh menu bên trái, truy cập mục **Subnet groups** $\rightarrow$ Nhấn nút **Create DB subnet group**.
3. Cấu hình các thông số:
   * **Name:** Nhập `enggo-db-subnet-group`.
   * **Description:** Nhập `Subnet group for EngGo RDS`.
   * **VPC:** Chọn `enggo-vpc`.
4. Tại mục **Add subnets**:
   * **Availability Zones:** Chọn 2 vùng khả dụng: `ap-northeast-1a` và `ap-northeast-1c`.
   * **Subnets:** Chọn 2 dải IP của Private Subnet: `10.0.10.0/24` (`enggo-private-subnet-1`) và `10.0.20.0/24` (`enggo-private-subnet-2`).
5. Nhấn **Create** để hoàn tất.

![Khởi tạo DB Subnet Group trên RDS Console](IMAGES/step1-create-db-subnet-group.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình trang tạo DB Subnet Group với tên enggo-db-subnet-group và chọn 2 Private Subnets.*

---

### Bước 2: Khởi tạo MySQL Database Instance (`enggo-db-instance`)

Tiến hành khởi tạo instance cơ sở dữ liệu MySQL trên hạ tầng đã được cấu hình.

#### Các bước thực hiện:
1. Tại menu bên trái RDS Console, chọn **Databases** $\rightarrow$ Nhấn **Create database**.
2. **Phương thức & Engine:**
   * **Creation method:** Chọn `Standard create`.
   * **Engine type:** Chọn `MySQL`.
   * **Engine Version:** Chọn phiên bản MySQL 8.0.x (Ví dụ: `MySQL 8.0.35`).
   * **Templates:** Chọn `Free tier` (Tối ưu chi phí).
3. **Cấu hình định danh & Tài khoản (Settings):**
   * **DB instance identifier:** Nhập `enggo-db-instance`.
   * **Master username:** Nhập `admin`.
   * **Auto generate a password:** Bỏ chọn.
   * **Master password / Confirm password:** Nhập mật khẩu quản trị an toàn (Ví dụ: `EngGoPass123!`).

![Cấu hình Settings cho RDS Instance](IMAGES/step2-rds-settings.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình cài đặt DB Instance Identifier, Master Username và Password.*

4. **Cấu hình Phần cứng & Lưu trữ (Instance & Storage):**
   * **DB instance class:** Chọn `db.t3.micro` (Thuộc gói Free Tier).
   * **Storage type:** Chọn `gp2` hoặc `gp3`.
   * **Allocated storage:** Nhập `20` GiB.
   * **Enable storage autoscaling:** Bỏ chọn (Tránh tự động tăng dung lượng gây phát sinh chi phí).
5. **Cấu hình Kết nối mạng (Connectivity) - Quan trọng:**
   * **Compute resource:** Chọn `Don’t connect to an EC2 compute resource`.
   * **Network type:** Chọn `IPv4`.
   * **Virtual private cloud (VPC):** Chọn `enggo-vpc`.
   * **DB subnet group:** Chọn `enggo-db-subnet-group` vừa tạo ở Bước 1.
   * **Public access:** Chọn **`No`** (Bắt buộc để đảm bảo an toàn tuyệt đối).
   * **VPC security group (firewall):** Chọn `Choose existing` $\rightarrow$ Gỡ bỏ group `default`, chọn đúng **`enggo-rds-sg`**.
   * **Availability Zone:** Chọn `ap-northeast-1a` (Hoặc `No preference`).

![Cấu hình Connectivity cho RDS](IMAGES/step2-rds-connectivity.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình thiết lập VPC, Public Access = No và Security Group enggo-rds-sg.*

6. **Cấu hình Bổ sung (Additional Configuration):**
   * Mở rộng mục cuối trang, tại **Initial database name**: Nhập `enggo_db` (RDS sẽ tự động tạo sẵn database này).
   * **Enable automated backups:** Có thể bỏ chọn hoặc đặt 1 ngày để tiết kiệm tài nguyên lưu trữ.
7. Nhấn **Create database**. Quá trình khởi tạo diễn ra trong khoảng 5 - 10 phút đến khi cột **Status** chuyển sang trạng thái `Available`.

---

### Bước 3: Thu thập thông tin Endpoint kết nối

Sau khi khởi tạo thành công, RDS sẽ cấp một đường dẫn DNS nội bộ (**Endpoint**) làm địa chỉ truy vấn cho ứng dụng.

#### Các bước thực hiện:
1. Tại giao diện danh sách **Databases**, nhấp vào tên instance `enggo-db-instance`.
2. Truy cập tab **Connectivity & security**.
3. Tại mục **Endpoint & port**, sao chép chuỗi Endpoint được tạo ra.
   * *Định dạng Endpoint:* `enggo-db-instance.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com`
   * *Cổng kết nối (Port):* `3306`

![Tab Connectivity & Security hiển thị Endpoint của RDS](IMAGES/step3-rds-endpoint.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình vị trí chuỗi Endpoint và Port 3306 trên RDS Console.*

---

### Bước 4: Tích hợp thông số RDS vào Dự án Spring Boot

Cập nhật tệp cấu hình nguồn của ứng dụng `enggo-backend` để kết nối trực tiếp đến cơ sở dữ liệu trên AWS.

#### Cấu hình tệp `src/main/resources/application.yml`:

```yaml
spring:
  datasource:
    url: jdbc:mysql://[enggo-db-instance.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com:3306/enggo_db?useSSL=true&requireSSL=false&serverTimezone=UTC](https://enggo-db-instance.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com:3306/enggo_db?useSSL=true&requireSSL=false&serverTimezone=UTC)
    username: admin
    password: EngGoPass123!
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQLDialect

```

{{% notice tip %}}
**Lưu ý khi triển khai thực tế (Production Best Practice):**
Không nên hardcode username và password trực tiếp vào file code `application.yml`. Thay vào đó, hãy sử dụng **Biến môi trường (Environment Variables)** hoặc các dịch vụ quản lý bí mật như **AWS Secrets Manager** / **GitHub Secrets** khi thiết lập pipeline CI/CD.
{{% /notice %}}

---

## 3. TỔNG KẾT VÀ NGUYÊN TẮC VẬN HÀNH BẢO MẬT

| Thành phần CSDL | Giá trị thiết lập | Cơ chế bảo mật đạt được |
| --- | --- | --- |
| **Vùng mạng** | `enggo-private-subnet-1` & `2` | Cách ly hoàn toàn khỏi Internet; không thể ping hay truy cập trực tiếp từ máy cá nhân |
| **Public Access** | `No` | Không cấp phát Public IP/DNS Public, triệt tiêu nguy cơ tấn công dò quét cổng từ bên ngoài |
| **Security Group** | `enggo-rds-sg` | Chỉ chấp nhận luồng TCP cổng 3306 phát ra từ các tài nguyên gán `enggo-app-sg` |
| **Tích hợp App** | Spring Boot Container (ECS/EC2) | Kết nối nội bộ qua mạng VPC với độ trễ cực thấp (Low Latency) và độ tin cậy cao |

> **Cảnh báo vận hành:** Do RDS đặt trong Private Subnet và chặn Public Access, các công cụ quản trị CSDL như MySQL Workbench hay DBeaver trên máy tính cá nhân sẽ **không thể kết nối trực tiếp**. Để quản trị dữ liệu thủ công, kỹ sư vận hành cần sử dụng kỹ thuật **SSH Tunneling (Bastion Host)** thông qua một máy chủ EC2 trung gian đặt ở Public Subnet.

```

```
Dưới đây là nội dung báo cáo chi tiết cho phần **Tích hợp Lưu trữ Đối tượng với Amazon S3**, được trình bày bằng định dạng **Markdown** chuẩn. Nội dung đã được chuẩn hóa vị trí chèn ảnh minh họa, đồng bộ vùng AWS Region với toàn bộ hệ thống (`ap-northeast-1` - Tokyo) và định dạng mã nguồn sắc nét.

Bạn chỉ cần sao chép (copy) đoạn Markdown dưới đây dán tiếp vào file tài liệu báo cáo `.md` của mình.

---


# TÍCH HỢP DỊCH VỤ LƯU TRỮ ĐỐI TƯỢNG AMAZON S3 VÀO ỨNG DỤNG ENGGO

---

## 1. TỔNG QUAN HẠ TẦNG LƯU TRỮ MEDIA (AMAZON S3)

Trong hệ thống **EngGo**, tài nguyên truyền thông bao gồm: ảnh đại diện người dùng (avatars), tệp âm thanh phát âm (audio lessons), hình ảnh bài thi và tài liệu học tập được quản lý tập trung trên dịch vụ lưu trữ đối tượng **Amazon Simple Storage Service (Amazon S3)**.

Ứng dụng Amazon S3 đem lại khả năng mở rộng không giới hạn (Scalability), độ tin cậy lưu trữ lên đến 99.999999999% (11 số 9) và khả năng truy cập trực tiếp từ phía Client với độ trễ cực thấp.

![Mô hình luồng tải và truy xuất tệp Media trên Amazon S3](IMAGES/s3-architecture-diagram.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh sơ đồ mô tả luồng Upload/Download tệp media giữa Frontend, Spring Boot Backend (AWS SDK v2) và Amazon S3 Bucket.*

### Đặc điểm thiết lập Kiến trúc:
* **Storage Bucket:** Khởi tạo `enggo-media-bucket-2026` tại vùng **Tokyo (`ap-northeast-1`)**.
* **Phân quyền truy cập công khai (Public Read):** Cấu hình Bucket Policy cho phép người dùng cuối (Frontend/Client) tải và hiển thị trực tiếp tài nguyên hình ảnh/âm thanh thông qua đường dẫn URL công khai.
* **Quyền truy xuất cấp ứng dụng (Programmatic Access):** Khởi tạo IAM User `enggo-s3-user` mang chính sách `AmazonS3FullAccess` cấp riêng cho ứng dụng Backend Spring Boot thực hiện các thao tác Upload/Delete file.

---

## 2. QUY TRÌNH THỰC HIỆN CHI TIẾT TRÊN AWS CONSOLE

### Bước 1: Khởi tạo Amazon S3 Bucket (`enggo-media-bucket-2026`)

1. Đăng nhập vào **AWS Management Console** $\rightarrow$ Tìm kiếm và chọn dịch vụ **S3**.
2. Nhấn nút **Create bucket**.
3. Cấu hình thông số chi tiết:
   * **Bucket name:** Nhập `enggo-media-bucket-2026` *(Tên duy nhất trên toàn hệ thống AWS)*.
   * **AWS Region:** Chọn `ap-northeast-1` (Tokyo).
   * **Object Ownership:** Chọn `ACLs disabled (recommended)`.
   * **Block Public Access settings for this bucket:** 
     * **Bỏ tích** mục `Block all public access` để cho phép phân phối file tĩnh công khai.
     * Tích chọn ô xác nhận nhận thức rủi ro: *"I acknowledge that the current settings might result in this bucket and the objects within it becoming public"*.
4. Các thông số khác giữ nguyên mặc định $\rightarrow$ Nhấn **Create bucket**.

![Tạo mới S3 Bucket trên AWS Console](IMAGES/step1-create-s3-bucket.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình cài đặt Bucket name, AWS Region và cấu hình Block Public Access.*

---

### Bước 2: Cấu hình Bucket Policy (Cấp quyền đọc công khai)

Để Frontend và thiết bị di động có thể hiển thị trực tiếp ảnh/audio qua URL, cần gán chính sách đọc công khai (`s3:GetObject`) cho tất cả các đối tượng trong Bucket.

1. Nhấp vào tên Bucket `enggo-media-bucket-2026` $\rightarrow$ Chuyển sang tab **Permissions**.
2. Tìm đến mục **Bucket policy** $\rightarrow$ Nhấn **Edit**.
3. Dán đoạn mã JSON cấu hình bên dưới:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::enggo-media-bucket-2026/*"
        }
    ]
}

```

4. Nhấn **Save changes**.


*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình trình biên soạn Bucket Policy với chính sách PublicReadGetObject.*

---

### Bước 3: Cấu hình CORS (Cross-Origin Resource Sharing)

Cấu hình CORS cho phép các yêu cầu HTTP (Upload/Fetch file) từ các tên miền Frontend (React, Mobile App) được phép tương tác với S3 Bucket.

1. Tại tab **Permissions**, kéo xuống mục **Cross-origin resource sharing (CORS)** $\rightarrow$ Nhấn **Edit**.
2. Dán đoạn cấu hình JSON:

```json
[
    {
        "AllowedHeaders": ["*"],
        "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
        "AllowedOrigins": ["*"],
        "ExposedHeaders": []
    }
]

```

3. Nhấn **Save changes**.


*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình cài đặt CORS JSON thành công.*

---

### Bước 4: Khởi tạo IAM User & Đã cấp Key truy cập cho Backend

Tạo định danh lập trình riêng biệt cho dịch vụ Backend `enggo-backend` để thao tác an toàn với S3.

1. Mở dịch vụ **IAM** trên AWS Console $\rightarrow$ Chọn **Users** $\rightarrow$ Nhấn **Create user**.
2. **User name:** Nhập `enggo-s3-user` $\rightarrow$ Nhấn **Next**.
3. **Permissions options:** Chọn `Attach policies directly`.
4. Tìm kiếm và chọn chính sách **`AmazonS3FullAccess`** $\rightarrow$ Nhấn **Next** $\rightarrow$ Nhấn **Create user**.
5. Nhấp vào User `enggo-s3-user` vừa tạo $\rightarrow$ Chuyển sang tab **Security credentials**.
6. Tại mục **Access keys** $\rightarrow$ Nhấn **Create access key**.
7. Chọn mục mục đích: `Application running outside AWS` $\rightarrow$ Nhấn **Next** $\rightarrow$ Nhấn **Create access key**.
8. **Lưu trữ bảo mật:** Sao chép hai chuỗi khóa thông tin:
* **Access Key ID:** *(Ví dụ: `AKIAIOSFODNN7EXAMPLE`)*
* **Secret Access Key:** *(Ví dụ: `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`)*




*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình bảng hiển thị Access Key ID và Secret Access Key.*

---

## 3. TÍCH HỢP AWS SDK VÀO DỰ ÁN SPRING BOOT

### 1. Khai báo Dependency (`pom.xml`)

Dự án tích hợp bộ thư viện chính thức **AWS SDK for Java v2 (S3 Module)**:

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
</dependency>

```

---

### 2. Cấu hình Tham số Hệ thống (`application.yml`)

Cập nhật thông số kết nối S3 vào tệp cấu hình nguồn:

```yaml
aws:
  s3:
    bucket-name: enggo-media-bucket-2026
    region: ap-northeast-1
    access-key: ${AWS_ACCESS_KEY_ID:<YOUR_ACCESS_KEY_ID>}
    secret-key: ${AWS_SECRET_ACCESS_KEY:<YOUR_SECRET_ACCESS_KEY>}

```

{{% notice info %}}
**Khuyến nghị Bảo mật:** Để tránh rò rỉ Credential trên Git repository, các tham số `access-key` và `secret-key` được cấu hình đọc trực tiếp từ **Biến môi trường (Environment Variables)** hệ thống khi chạy trên môi trường Docker/EC2/ECS.
{{% /notice %}}

---

### 3. Cấu hình Bean `S3Client` (`S3Config.java`)

Tạo Lớp Cấu hình (Configuration Class) để đăng ký `S3Client` vào Spring Application Context.

```java
package com.nhom12.enggo_backend.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import software.amazon.awssdk.auth.credentials.AwsBasicCredentials;
import software.amazon.awssdk.auth.credentials.StaticCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;

@Configuration
public class S3Config {

    @Value("${aws.s3.region}")
    private String region;

    @Value("${aws.s3.access-key}")
    private String accessKey;

    @Value("${aws.s3.secret-key}")
    private String secretKey;

    @Bean
    public S3Client s3Client() {
        AwsBasicCredentials credentials = AwsBasicCredentials.create(accessKey, secretKey);
        
        return S3Client.builder()
                .region(Region.of(region))
                .credentialsProvider(StaticCredentialsProvider.create(credentials))
                .build();
    }
}

```

---

### 4. Xây dựng Service Tải tệp (`S3Service.java`)

Lớp nghiệp vụ chịu trách nhiệm tiếp nhận file upload (`MultipartFile`), tạo định danh file duy nhất (dùng `UUID`) và truyền tải dòng dữ liệu (InputStream) lên Amazon S3.

```java
package com.nhom12.enggo_backend.service;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import software.amazon.awssdk.core.sync.RequestBody;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.PutObjectRequest;

import java.io.IOException;
import java.util.UUID;

@Service
public class S3Service {

    private final S3Client s3Client;

    @Value("${aws.s3.bucket-name}")
    private String bucketName;

    @Value("${aws.s3.region}")
    private String region;

    public S3Service(S3Client s3Client) {
        this.s3Client = s3Client;
    }

    public String uploadFile(MultipartFile file) throws IOException {
        // Tạo tên tệp chuẩn hóa duy nhất tránh ghi đè dữ liệu
        String fileName = UUID.randomUUID() + "_" + file.getOriginalFilename();

        PutObjectRequest putObjectRequest = PutObjectRequest.builder()
                .bucket(bucketName)
                .key(fileName)
                .contentType(file.getContentType())
                .build();

        // Đẩy Stream tệp lên Amazon S3 Bucket
        s3Client.putObject(putObjectRequest, RequestBody.fromInputStream(file.getInputStream(), file.getSize()));

        // Trả về định dạng đường dẫn công khai (Public Virtual-Hosted URL)
        return String.format("https://%s.s3.%[s.amazonaws.com/%s](https://s.amazonaws.com/%s)", bucketName, region, fileName);
    }
}

```

---

## 4. BẢNG TỔNG KẾT CẤU HÌNH S3 MULTIMEDIA

| Tham số Cấu hình | Giá trị Thực tế | Vai trò trong Hệ thống |
| --- | --- | --- |
| **S3 Bucket Name** | `enggo-media-bucket-2026` | Lưu trữ tập trung toàn bộ media của ứng dụng EngGo |
| **AWS Region** | `ap-northeast-1` (Tokyo) | Tối ưu tốc độ truyền tải tệp về thị trường Việt Nam / Đông Á |
| **Access Model** | Public Read Object via Policy | Cho phép Client xem trực tiếp ảnh/audio mà không qua proxy backend |
| **Authentication** | IAM User `enggo-s3-user` | Cấp quyền lập trình (Programmatic Access) giới hạn cho Spring Boot |
| **SDK Integration** | AWS SDK for Java v2 (`software.amazon.awssdk:s3`) | Tối ưu hiệu năng I/O và hỗ trợ các chuẩn AWS modern API |

```

```