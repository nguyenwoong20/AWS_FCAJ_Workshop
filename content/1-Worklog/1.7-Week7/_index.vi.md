---
title: "Nhật ký tuần 7"
date: 2026-06-10
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

{{% notice note %}}
✏️ Bản nháp — hãy chỉnh lại theo công việc thực tế của bạn trong tuần này (thêm ngày tháng, chi tiết, link tài liệu).
{{% /notice %}}

### Mục tiêu tuần 7

- Hiện thực Lambda phim theo chiến lược drop-in replacement cho API Express cũ.

### Kết quả đạt được tuần 7

- Phân tích cách app Flutter gọi API cũ (route, format response, chi tiết theo slug) và chốt chiến lược drop-in replacement.
- Viết handler cinemax-movies: /api/movies/limit, /category/{slug}, /country/{slug}, /year, chi tiết /{slug}, tìm kiếm — tất cả trả {success, data} như Express.
- Thêm GSI slug-index để màn hình chi tiết tra theo slug bằng Query thay vì Scan.
- Thiết kế item hai lớp: trường nhẹ top-level cho danh sách (ProjectionExpression), document gốc đầy đủ trong thuộc tính `doc` cho chi tiết.
