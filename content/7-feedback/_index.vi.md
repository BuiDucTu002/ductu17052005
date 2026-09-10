---
title: "Chia sẻ, đóng góp ý kiến"
pre: "<b>7. </b>"
date: 2026-09-10
weight: 7
chapter: false
---

# 7. CHIA SẺ, ĐÓNG GÓP Ý KIẾN

### 7.1. Bài học kinh nghiệm thu được (Key Takeaways)

1. **Tư duy Bảo mật Đám mây (Cloud Security First):** Việc tuân thủ nguyên tắc quyền tối thiểu (Least Privilege) và phân tầng Security Group (SG-to-SG Referencing) giúp hệ thống được bảo vệ vững chắc ngay từ khâu thiết kế ban đầu.
2. **Sức mạnh của Tự động hóa (CI/CD Power):** Việc tự động hóa quy trình đóng gói và triển khai giúp giảm thiểu sai sót do con người, tăng tốc độ thử nghiệm tính năng mới và giữ cho môi trường Staging/Production luôn đồng bộ.
3. **Quản lý Chi phí trên AWS (FinOps & Cost Control):** Cần luôn chú ý đến việc tắt/xóa các tài nguyên không dùng đến (như Elastic IP không gắn vào EC2, Snapshot thừa, DB Instance nhàn rỗi) và ưu tiên sử dụng các gói Free Tier (`t3.micro`, `db.t3.micro`) trong giai đoạn học tập, nghiên cứu.

---

### 7.2. Ý kiến đóng góp & Đề xuất

* **Đối với Chương trình Học tập / Môn học:**
  * Đề xuất tăng cường các bài thực hành Lab thực tế về mô hình triển khai Microservices hóa trên Docker/Kubernetes kết hợp Cloud.
  * Hướng dẫn chi tiết hơn về các công cụ Infrastructure as Code (IaC) như **Terraform** hoặc **AWS CDK** để sinh viên có thể khởi tạo toàn bộ hạ tầng VPC/RDS/EC2 tự động bằng mã nguồn thay vì thao tác thủ công trên Console.
* **Đối với Nền tảng AWS:**
  * Dịch vụ AWS Console cung cấp giao diện trực quan, trực diện và rất dễ thao tác đối với người mới bắt đầu. Tuy nhiên, thời gian tạo các tài nguyên như RDS khá lâu (5-10 phút), cần chuẩn bị kỹ kịch bản thực hiện để tối ưu thời gian làm bài.