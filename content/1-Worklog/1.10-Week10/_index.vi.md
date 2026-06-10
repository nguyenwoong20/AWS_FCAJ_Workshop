---
title: "Nhật ký tuần 10"
date: 2026-06-10
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

{{% notice note %}}
✏️ Bản nháp — hãy chỉnh lại theo công việc thực tế của bạn trong tuần này (thêm ngày tháng, chi tiết, link tài liệu).
{{% /notice %}}

### Mục tiêu tuần 10

- Kết nối app Flutter thật, test end-to-end trên máy thật, xác minh giám sát.

### Kết quả đạt được tuần 10

- Migrate app bằng cách đổi một hằng số trong api_config.dart — endpoint phim + auth giờ trỏ về API Gateway.
- Xử lý hai lỗi build có thật: thiếu google-services.json (config Firebase bị gitignore) và bug Kotlin incremental compiler khi project với pub cache nằm khác ổ đĩa (kotlin.incremental=false).
- Chạy app trên điện thoại Android thật với server cũ TẮT: duyệt phim, tìm kiếm, phát video, đăng ký/OTP/đăng nhập đều do AWS phục vụ.
- Xem request trực tiếp trong CloudWatch Logs, đo Duration/Memory từ dòng REPORT, và cố ý kích hoạt + khôi phục alarm 5XX.
