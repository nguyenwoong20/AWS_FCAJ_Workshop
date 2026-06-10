---
title: "Workshop"
date: 2026-06-10
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Migrate backend app Flutter thật sang AWS Serverless

Trong workshop này, bạn sẽ chuyển backend của **Cinemax** — một ứng dụng xem phim Flutter thực tế — từ server Node.js/Express + MongoDB truyền thống sang **kiến trúc serverless hoàn toàn trên AWS**, theo chiến lược *drop-in replacement*: API mới trên cloud mô phỏng chính xác route và format response của API cũ, nên app mobile chỉ cần đổi đúng một dòng base URL.

Kết thúc workshop, danh mục phim, tìm kiếm, dữ liệu phát phim **và toàn bộ hệ thống xác thực** (đăng ký, OTP qua email, đăng nhập, Google Sign-In, đặt lại mật khẩu) của app đều chạy trên AWS — không server nào phải bảo trì, chi phí ≈ $0/tháng.

![Kiến trúc Cinemax Serverless](/images/2-Proposal/architecture.svg)

### Nội dung

1. [Tổng quan Workshop](5.1-workshop-overview/)
2. [Chuẩn bị](5.2-prerequiste/)
3. [Triển khai Backend Serverless](5.3-deploy-backend/)
4. [Di chuyển dữ liệu từ MongoDB](5.4-data-migration/)
5. [Xác thực Serverless](5.5-auth-api/)
6. [Kết nối app Flutter & Kiểm thử](5.6-test-app/)
7. [Dọn dẹp tài nguyên](5.7-cleanup/)
