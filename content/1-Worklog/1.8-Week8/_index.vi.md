---
title: "Nhật ký tuần 8"
date: 2026-06-10
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

{{% notice note %}}
✏️ Bản nháp — hãy chỉnh lại theo công việc thực tế của bạn trong tuần này (thêm ngày tháng, chi tiết, link tài liệu).
{{% /notice %}}

### Mục tiêu tuần 8

- Di chuyển dữ liệu phim thật từ MongoDB và vượt qua giới hạn item 400 KB.

### Kết quả đạt được tuần 8

- Export 240 phim từ MongoDB (mongoexport) và viết scripts/seed.js nhập theo lô 25 (giới hạn BatchWriteItem).
- Gặp ValidationException thật: phim bộ dài tập vượt giới hạn 400 KB/item của DynamoDB — MongoDB không hề phàn nàn vì giới hạn của nó là 16 MB.
- Sửa bằng nén lũy tiến: bỏ trường tập phim không thiết yếu trước, sau đó chỉ giữ server phát đầu tiên; phát phim vẫn nguyên vẹn.
- Kiểm tra query slug-index và thể loại từ CLI trên dữ liệu đã seed.
