---
title: "Xóa tài nguyên AWS"
weight: 1
chapter: false
pre: "<b>5.6.1. </b>"
date: 2026-09-10
---

## 5.6. Dọn dẹp tài nguyên

### 5.6.1. Xóa tài nguyên AWS

Để tránh phát sinh chi phí duy trì tài nguyên không mong muốn trên hệ thống đám mây Amazon Web Services (AWS) sau khi hoàn tất quá trình kiểm thử và đánh giá dự án, toàn bộ hạ tầng đã khởi tạo cần được giải phóng và xóa bỏ theo một quy trình chuẩn hóa.

Quy trình hủy bỏ tài nguyên cần thực hiện theo thứ tự ngược lại với quy trình khởi tạo (từ các dịch vụ cao cấp ứng dụng xuống dần hạ tầng mạng cơ sở) nhằm tránh lỗi vi phạm phụ thuộc giữa các tài nguyên (Resource Dependency Errors).

---

#### Quy trình chi tiết xóa bỏ tài nguyên AWS:

##### Bước 1: Xóa máy chủ Amazon EC2 Instance
1. Truy cập **EC2 Console** $\rightarrow$ chọn mục **Instances**.
2. Tích chọn máy chủ `enggo-backend-server`.
3. Nhấn **Instance state** $\rightarrow$ Chọn **Terminate instance**.
4. Xác nhận xóa. *(Thao tác này sẽ tự động giải phóng Public IP và xóa ổ đĩa EBS đính kèm)*.

![Châm dứt và xóa EC2 Instance trên Console](IMAGES/cleanup-step1-ec2-terminate.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình thao tác chọn Terminate instance cho enggo-backend-server.*

---

##### Bước 2: Xóa Cơ sở dữ liệu Amazon RDS MySQL
1. Truy cập **RDS Console** $\rightarrow$ chọn **Databases**.
2. Chọn `enggo-db-instance` $\rightarrow$ Nhấn **Actions** $\rightarrow$ Chọn **Delete**.
3. Tại trang xác nhận:
   * **Bỏ tích** mục *Create final snapshot?* (Tránh lưu lại bản sao lưu gây tính phí S3 Storage).
   * Tích chọn ô *I acknowledge that upon deletion of this database...*.
   * Gõ từ khóa `delete me` vào ô xác nhận $\rightarrow$ Nhấn **Delete**.
4. Vào mục **Subnet groups** $\rightarrow$ Tích chọn `enggo-db-subnet-group` $\rightarrow$ Nhấn **Delete**.

![Thao tác xóa RDS Instance trên Console](IMAGES/cleanup-step2-rds-delete.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình trang xác nhận xóa RDS Instance với ô nhập delete me.*

---

##### Bước 3: Xóa Amazon ECR Repository
1. Truy cập **ECR Console** $\rightarrow$ chọn **Repositories**.
2. Tích chọn kho chứa `enggo-backend`.
3. Nhấn nút **Delete** $\rightarrow$ Nhập `delete` vào ô xác nhận $\rightarrow$ Nhấn **Delete**. *(Thao tác này sẽ xóa toàn bộ các Docker Images đã lưu trữ trong kho)*.

---

##### Bước 4: Dọn dẹp tệp tin và Xóa Amazon S3 Bucket
1. Truy cập **S3 Console** $\rightarrow$ nhấp chọn `enggo-media-bucket-2026`.
2. Nhấn nút **Empty** để xóa sạch toàn bộ các tệp tin/media đang chứa bên trong Bucket $\rightarrow$ Nhập `permanently delete` để xác nhận.
3. Sau khi Bucket đã rỗng, quay lại danh sách Bucket $\rightarrow$ Tích chọn `enggo-media-bucket-2026` $\rightarrow$ Nhấn **Delete** $\rightarrow$ Nhập tên Bucket để xác nhận xóa hoàn toàn.

![Xóa sạch dữ liệu và hủy bỏ S3 Bucket](IMAGES/cleanup-step4-s3-delete.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình thao tác Empty và Delete S3 Bucket.*

---

##### Bước 5: Xóa IAM User và Access Keys
1. Truy cập **IAM Console** $\rightarrow$ chọn **Users** $\rightarrow$ nhấp chọn `enggo-s3-user`.
2. Tại tab **Security credentials**, chọn và xóa tất cả các **Access Keys** hiện có.
3. Quay lại danh sách Users $\rightarrow$ Chọn `enggo-s3-user` $\rightarrow$ Nhấn **Delete**.

---

##### Bước 6: Xóa Hạ tầng mạng VPC (`enggo-vpc`)
Sau khi đã giải phóng toàn bộ các tài nguyên tính toán và lưu trữ nằm bên trong VPC, tiến hành hủy bỏ hạ tầng mạng ảo:

1. Truy cập **VPC Console** $\rightarrow$ chọn **Your VPCs**.
2. Tích chọn `enggo-vpc` $\rightarrow$ Nhấn **Actions** $\rightarrow$ Chọn **Delete VPC**.
3. Hệ thống AWS sẽ tự động phân tích và hiển thị danh sách tất cả các thành phần phụ thuộc liên quan đến VPC này (bao gồm: *4 Subnets, Internet Gateway `enggo-igw`, Route Tables và các Security Groups `enggo-app-sg`, `enggo-rds-sg`, `enggo-ec2-sg`*).
4. Nhập `delete` vào ô xác nhận $\rightarrow$ Nhấn **Delete**. Hệ thống sẽ tự động giải phóng hoàn toàn toàn bộ hạ tầng mạng liên quan trong một thao tác duy nhất.

![Xóa VPC và toàn bộ tài nguyên mạng phụ thuộc](IMAGES/cleanup-step6-vpc-delete.png)
*> **[GHI CHÚ CHÈN ẢNH]:** Chèn ảnh chụp màn hình bảng xác nhận Delete VPC hiển thị các Subnets, IGW và Security Groups liên đới được giải phóng.*

---

#### Bảng tổng hợp kiểm kê trạng thái dọn dẹp tài nguyên

| Tài nguyên AWS | Tên định danh tài nguyên | Trạng thái sau dọn dẹp | Mục đích kiểm soát chi phí |
| :--- | :--- | :---: | :--- |
| **Amazon EC2** | `enggo-backend-server` | **Terminated** | Tránh phát sinh chi phí vCPU / RAM và ổ đĩa EBS Storage |
| **Amazon RDS** | `enggo-db-instance` | **Deleted** | Tránh phát sinh chi phí DB Instance hour và DB Storage |
| **Amazon ECR** | `enggo-backend` | **Deleted** | Tránh tính phí lưu trữ Docker Images theo dung lượng MB |
| **Amazon S3** | `enggo-media-bucket-2026` | **Deleted** | Giải phóng dung lượng lưu trữ đối tượng (Object Storage) |
| **Amazon VPC** | `enggo-vpc` | **Deleted** | Giải phóng dải IP và các cấu hình định tuyến ảo |