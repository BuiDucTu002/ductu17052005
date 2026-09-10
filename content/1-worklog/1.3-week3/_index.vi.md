---
title: "1.3. Tuần 3 worklog"
date: "`r Sys.Date()`"
weight: 3
chapter: false
---

# Tuần 3: Phát triển Backend và Containerization

### Mục tiêu tuần 3:

- Cấu hình Spring Boot application
- Thiết lập Spring Data JPA và database mappings
- Phát triển API endpoints
- Cấu hình Docker cho ứng dụng
- Build và test Docker Image cục bộ
- Tối ưu hóa kích thước Docker Image

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|------|-----------|--------------|----------------|-------------------|
| 2 | Cấu hình Spring Boot properties | 27/04/2026 | 27/04/2026 | IntelliJ IDEA |
| 3 | Thiết lập Spring Data JPA và mappings | 28/04/2026 | 28/04/2026 | Spring Documentation |
| 4 | Phát triển REST API endpoints | 29/04/2026 | 29/04/2026 | Spring Boot Guide |
| 5 | Tạo Dockerfile và build Docker Image | 30/04/2026 | 30/04/2026 | Docker Documentation |
| 6 | Test Docker Image cục bộ và tối ưu | 01/05/2026 | 01/05/2026 | Docker Best Practices |

### Kết quả đạt được tuần 3:

**Cấu hình Spring Boot:**
- Cấu hình thành công Spring Boot application với AWS RDS connection
- Thiết lập application.yml với environment-specific properties
- Cấu hình Spring Data JPA với Hibernate ORM
- Tạo database entity classes với proper annotations

**Phát triển API:**
- Phát triển complete REST API endpoints cho core functionality
- Triển khai Spring Security cho authentication
- Thiết lập JWT token-based authorization
- Tạo request/response DTOs với validation

**Spring Data JPA:**
- Tạo repository interfaces cho database operations
- Triển khai custom query methods
- Cấu hình relationship mappings (One-to-Many, Many-to-Many)
- Thiết lập lazy loading configuration

**Docker Implementation:**
- Tạo optimized Dockerfile với multi-stage build
- Cấu hình Docker entrypoint cho application startup
- Thiết lập environment variables để cấu hình
- Tối ưu hóa kích thước Docker Image từ ~800MB xuống ~250MB

**Test Cục bộ:**
- Build Docker Image thành công trên development machine
- Chạy Docker container cục bộ với RDS connection
- Test API endpoints bằng curl và Postman
- Xác thực database operations từ containerized application
- Sửa lỗi kết nối và tối ưu performance

