---
title: "Đề xuất dự án"
date: 2026-06-10
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Cinemax Serverless API
## Tái kiến trúc backend ứng dụng xem phim bằng AWS Serverless

### 1. Tóm tắt

Cinemax là ứng dụng xem phim trên di động xây dựng bằng Flutter mà tôi đã phát triển trước đây. Backend hiện tại chạy trên Node.js/Express với MongoDB, phải thuê server chạy liên tục — tốn chi phí ngay cả khi không có người dùng và không thể tự động mở rộng khi lượng truy cập tăng đột biến.

Dự án này tái kiến trúc phần lõi của backend Cinemax (danh mục phim và bookmark của người dùng) sang **kiến trúc serverless hoàn toàn trên AWS**, sử dụng Amazon API Gateway, AWS Lambda, Amazon DynamoDB, Amazon S3 và Amazon CloudWatch. Kết quả là một backend gần như miễn phí ở mức truy cập thấp, tự động mở rộng và không cần bảo trì server.

### 2. Vấn đề cần giải quyết

#### Vấn đề là gì?

- Backend Express + MongoDB hiện tại cần VPS hoặc dịch vụ hosting chạy **24/7**, kể cả khi app không có người dùng.
- Việc mở rộng phải làm thủ công (nâng cấp server, cấu hình load balancer).
- Không có giám sát hay cảnh báo tự động; khi API lỗi, không ai biết cho đến khi người dùng phàn nàn.
- Ảnh poster phim đang lấy từ URL bên thứ ba, có thể hỏng bất cứ lúc nào.

#### Giải pháp

Chuyển API lõi sang các dịch vụ serverless của AWS:

- **Amazon API Gateway** cung cấp REST API.
- **AWS Lambda** (Node.js 22) chỉ chạy code khi có request — trả tiền theo lần gọi.
- **Amazon DynamoDB** (chế độ on-demand) lưu phim và bookmark — trả tiền theo request.
- **Amazon S3** lưu trữ ảnh poster phim ổn định.
- **Amazon CloudWatch** thu thập log từ mọi lần gọi Lambda và phát cảnh báo khi API trả lỗi 5XX.

### 3. Kiến trúc giải pháp

![Kiến trúc Cinemax Serverless](/images/2-Proposal/architecture.svg)

#### Các dịch vụ AWS sử dụng (5)

| Dịch vụ | Vai trò |
|---|---|
| Amazon API Gateway | Endpoint REST API cho app Flutter (`/api/movies/*`, `/api/auth/*`, bookmarks) |
| AWS Lambda | Ba function: `cinemax-movies` (danh mục + tìm kiếm), `cinemax-auth` (đăng ký/đăng nhập/OTP/Google) và `cinemax-bookmarks` |
| Amazon DynamoDB | Ba bảng: `cinemax-movies` (GSI: `slug-index`, `category-createdAt-index`), `cinemax-users` (mật khẩu băm bcrypt) và `cinemax-bookmarks` |
| Amazon S3 | Lưu ảnh poster, chỉ cho phép đọc công khai trên prefix `posters/` |
| Amazon CloudWatch | Log Lambda, metric API, cảnh báo lỗi 5XX |

#### Vì sao chọn các dịch vụ này?

- **Lambda + API Gateway**: không tốn chi phí khi nhàn rỗi, tự động scale, và tôi đã quen JavaScript/Node.js từ backend gốc.
- **DynamoDB on-demand**: free tier bao gồm 25 GB và hàng triệu request; không gặp vấn đề connection pooling như MongoDB chạy với Lambda.
- **S3**: độ bền dữ liệu 99,999999999% cho ảnh, ổn định hơn nhiều so với hot-link URL bên thứ ba.
- **CloudWatch**: tích hợp sẵn, không cần cài agent.
- **AWS SAM** được dùng làm Infrastructure-as-Code để toàn bộ stack có thể triển khai hoặc xóa chỉ bằng một lệnh.

### 4. Triển khai kỹ thuật

1. **Thiết kế** mô hình dữ liệu DynamoDB (bảng movies + GSI, bảng bookmarks với khóa tổ hợp `userId` + `movieId`).
2. **Phát triển** Lambda handler bằng Node.js với AWS SDK v3.
3. **Định nghĩa** toàn bộ tài nguyên trong SAM template (`template.yaml`) với IAM policy tối thiểu cho từng function.
4. **Triển khai** bằng `sam build && sam deploy`.
5. **Di chuyển dữ liệu**: script seed nhập dữ liệu phim thật xuất từ MongoDB gốc vào DynamoDB.
6. **Kiểm thử**: gọi mọi endpoint bằng curl/Postman, xem log trong CloudWatch, cố ý kích hoạt cảnh báo lỗi.
7. **Kết nối** app Flutter với base URL API mới.

### 5. Lộ trình & cột mốc

- **Tuần 1–3**: Học nền tảng AWS (IAM, Lambda, DynamoDB, S3, CloudWatch).
- **Tuần 4–5**: Thiết kế kiến trúc + viết proposal này.
- **Tuần 6–9**: Hiện thực: SAM template, code Lambda, di chuyển dữ liệu, poster lên S3.
- **Tuần 10–11**: Kiểm thử, thiết lập giám sát, viết tài liệu (workshop này).
- **Tuần 12**: Hoàn thiện báo cáo, dọn dẹp tài nguyên, thuyết trình.

### 6. Dự toán ngân sách

Với AWS Free Tier (12 tháng đầu):

| Dịch vụ | Ước tính sử dụng | Chi phí/tháng |
|---|---|---|
| Lambda | < 10.000 lần gọi | $0.00 (free tier: 1 triệu/tháng) |
| API Gateway | < 10.000 request | $0.00 (free tier: 1 triệu/tháng trong 12 tháng đầu) |
| DynamoDB | < 1 GB, on-demand | $0.00 (free tier: 25 GB) |
| S3 | ~500 MB poster | $0.02 |
| CloudWatch | log + 1 alarm | $0.00 (free tier: 10 alarm) |

**Tổng: ≈ $0.02/tháng** — gần như miễn phí trong suốt kỳ thực tập.

### 7. Đánh giá rủi ro

| Rủi ro | Mức độ ảnh hưởng | Xác suất | Biện pháp |
|---|---|---|---|
| Phát sinh chi phí AWS ngoài ý muốn | Trung bình | Thấp | Đặt AWS Budget cảnh báo ở mức $1; dùng on-demand; có hướng dẫn clean-up |
| Thiết kế sai mô hình dữ liệu DynamoDB | Trung bình | Trung bình | Thiết kế GSI từ đầu; test query trước khi di chuyển toàn bộ dữ liệu |
| Độ trễ cold start của Lambda | Thấp | Cao | Chấp nhận được với use case này (< 1s với arm64 + bundle nhỏ) |
| Hết hạn Free Tier | Thấp | Thấp | Dự án hoàn thành trong thời gian free tier |

### 8. Kết quả kỳ vọng

- API serverless hoạt động với dữ liệu phim thật của Cinemax.
- App Flutter chạy với backend AWS.
- Workshop step-by-step có thể tái lập, ai cũng triển khai lại được stack này.
- Giám sát bằng CloudWatch log và cảnh báo 5XX tự động.
- Tổng chi phí vận hành ≈ $0 và không phải bảo trì server nào.
