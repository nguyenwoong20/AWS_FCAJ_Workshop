---
title: "Nhật ký tuần 9"
date: 2026-06-10
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

{{% notice note %}}
✏️ Bản nháp — hãy chỉnh lại theo công việc thực tế của bạn trong tuần này (thêm ngày tháng, chi tiết, link tài liệu).
{{% /notice %}}

### Mục tiêu tuần 9

- Xây lại toàn bộ hệ thống xác thực theo hướng serverless.

### Kết quả đạt được tuần 9

- Hiện thực Lambda cinemax-auth với đủ 7 endpoint của backend cũ: register, verify-email (OTP), login, google-login, gửi lại OTP, quên/đặt lại mật khẩu.
- Gửi OTP email thật ngay trong Lambda bằng nodemailer + Gmail app password; secret truyền qua CloudFormation parameter NoEcho.
- Xác minh Google idToken phía server bằng google-auth-library; giữ nguyên JWT secret để token hiện có vẫn hợp lệ.
- Tạo bảng cinemax-users (PK: email) với mật khẩu băm bcrypt; test trọn luồng đăng ký → OTP → xác minh → đăng nhập bằng curl.
