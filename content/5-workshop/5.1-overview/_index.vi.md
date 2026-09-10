---
title: "Tổng quan Workshop"
weight: 1
chapter: true
pre: "<b>5.1. </b>"
date: 2026-09-10
---

# Tổng quan Workshop

Workshop hướng dẫn xây dựng, đóng gói và triển khai tự động backend EngGo lên AWS.

## Bối cảnh và bài toán

EngGo phục vụ người dùng học tiếng Anh, làm bài thi, đấu PvP và tính điểm ELO theo thời gian thực qua WebSocket. Hệ thống cần vận hành liên tục, lưu trữ audio và hình ảnh an toàn, tách biệt cơ sở dữ liệu khỏi ứng dụng web và loại bỏ deploy thủ công.

Đối tượng sử dụng là học sinh, sinh viên và các ứng dụng Frontend/Mobile gọi API.

## Kết quả đạt được

* REST API và WebSocket hoạt động tại cổng `8080`.
* Redis xử lý realtime state cho phòng chờ PvP.
* RDS MySQL hoạt động an toàn trong Private Subnet.
* Audio và avatar được lưu trữ trên Amazon S3.
* Docker Image được đẩy lên ECR và ứng dụng trên EC2 được cập nhật tự động khi push vào nhánh `main`.
