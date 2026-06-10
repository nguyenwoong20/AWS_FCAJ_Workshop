---
title: "Tổng quan Workshop"
date: 2026-06-10
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### Bối cảnh & bài toán

Cinemax là ứng dụng xem phim tôi xây dựng bằng Flutter. Backend của nó là server Node.js/Express với MongoDB, phải chạy trên máy cá nhân (hoặc thuê VPS):

- Server phải chạy **24/7** kể cả khi không có người dùng — lãng phí chi phí.
- App chỉ hoạt động khi máy tính/server đang bật.
- Không có giám sát: API chết không ai biết.
- Muốn scale phải nâng cấp server thủ công.

**Mục tiêu:** chuyển backend lên AWS serverless để app chạy mọi lúc mọi nơi, tự động scale, chi phí ≈ $0 ở mức truy cập sinh viên — **mà không phải viết lại app mobile**.

### Chiến lược then chốt: drop-in replacement

Thay vì thiết kế lại API (sẽ kéo theo sửa đổi khắp codebase Flutter), backend serverless mới **mô phỏng chính xác API Express cũ** — cùng route (`/api/movies/limit/{n}`, `/api/movies/category/{slug}`, `/api/auth/login`...), cùng format JSON response (`{success: true, data: [...]}`), cùng JWT secret.

Kết quả: migrate app = đổi **một hằng số** trong `api_config.dart`.

### Bạn sẽ xây dựng gì

| Thành phần | Tài nguyên AWS |
|---|---|
| API phim (danh sách, chi tiết theo slug, tìm kiếm, lọc thể loại/quốc gia/năm) | Lambda `cinemax-movies` + DynamoDB `cinemax-movies` (GSI: `slug-index`, `category-createdAt-index`) |
| Xác thực đầy đủ (đăng ký, OTP email, đăng nhập, Google Sign-In, đặt lại mật khẩu) | Lambda `cinemax-auth` + DynamoDB `cinemax-users` + Gmail SMTP |
| Bookmark người dùng | Lambda `cinemax-bookmarks` + DynamoDB `cinemax-bookmarks` |
| Định tuyến REST | Amazon API Gateway (`/api/movies/*`, `/api/auth/*`) |
| Lưu trữ poster | Amazon S3 (chỉ cho đọc công khai trên prefix `posters/`) |
| Giám sát | CloudWatch Logs + alarm lỗi 5XX |

Tất cả định nghĩa trong **một SAM template** (`template.yaml`) — Infrastructure-as-Code, triển khai và xóa chỉ với một lệnh.

### Bạn sẽ học được gì

- Tái kiến trúc backend Express + MongoDB thật sang Lambda + DynamoDB.
- Mô hình hóa DynamoDB từ access pattern thực tế (và vượt qua **giới hạn item 400 KB** với dữ liệu thật).
- Gửi OTP email và xác minh Google token ngay trong Lambda.
- IAM least-privilege: mỗi function chỉ truy cập được bảng của nó; không hard-code credential.
- Đọc log CloudWatch, test alarm, và kết nối điện thoại Android thật end-to-end.

### Thời lượng & chi phí

- **Thời lượng:** 2–3 giờ end-to-end.
- **Chi phí:** ≈ $0 với AWS Free Tier (xem [Proposal](../../2-proposal/)).
