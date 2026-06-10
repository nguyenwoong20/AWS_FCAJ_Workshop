---
title: "Chia sẻ và góp ý"
date: 2026-06-10
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

### Cảm nhận

Nói thật lòng: đây là dự án cá nhân tôi tự làm để học AWS một cách nghiêm túc — và nó vui hơn tôi tưởng rất nhiều.

Xuất phát điểm của tôi là một sinh viên HUTECH viết app Flutter, backend Express + MongoDB chạy trên máy cá nhân, tắt máy là app chết. Sau dự án này, app của tôi chạy trên hạ tầng của Amazon, server tắt vĩnh viễn mà app vẫn sống, và hóa đơn mỗi tháng rẻ hơn một gói mì tôm.

![Just a chill guy HUTECH](/images/7-Feedback/nh.jpg?width=350px)

*Trạng thái tinh thần xuyên suốt dự án: just a chill guy.*

Khoảnh khắc đáng nhớ nhất? Lúc seed 240 phim vào DynamoDB và màn hình đỏ rực `ValidationException: Item size has exceeded the maximum allowed size` lúc nửa đêm:

![Tôi sau khi debug](/images/7-Feedback/meo.png?width=400px)

*Tôi, 1 giờ sáng, sau khi gặp lỗi 400KB.*

Nhưng rồi cũng qua. Và cái cảm giác mở app trên điện thoại thật, tắt hẳn server Node, mà danh sách phim vẫn load vèo vèo từ Singapore về:

![SIUUU](/images/7-Feedback/cr7.jpg?width=400px)

*Khi `GET /api/movies` trả về 200 OK từ Lambda. SIUUU.*

### Mức độ hài lòng

**Rất hài lòng.** Đặc biệt là lúc mở trang Billing:

![Ngỡ ngàng](/images/7-Feedback/ngao.png?width=350px)

*Mặt tôi khi thấy tổng chi phí AWS: ~$0.02/tháng.*

Serverless đúng nghĩa là "trả tiền cho cái mình dùng" — và sinh viên thì dùng có bao nhiêu đâu.

### Điểm cần cải thiện (của chính tôi)

- Nên thiết kế mô hình dữ liệu DynamoDB **trước khi** migrate, thay vì vừa làm vừa sửa — đỡ phải seed lại 3 lần.
- Bookmark/comment vẫn còn trên backend cũ — mục tiêu tiếp theo là dọn nốt lên AWS.
- Đặt Budget alert **trước** khi deploy, đừng để deploy xong mới nhớ ra.

![Sigma](/images/7-Feedback/sigma.jpg?width=400px)

*Tinh thần khi quyết định khai tử server Express chạy 24/7.*

### Có giới thiệu cho bạn bè không? Vì sao?

**Có, 100%.** Nếu bạn là sinh viên IT có sẵn một project cá nhân (app, web gì cũng được), việc migrate nó lên AWS serverless là cách học cloud thực chiến nhất tôi từng thử — hơn hẳn xem 10 tiếng tutorial. Bạn sẽ gặp lỗi thật, đọc log thật, và cuối cùng có một sản phẩm thật để khoe.

![Đi khoe project](/images/7-Feedback/rua-shopping.png?width=350px)

*Tôi, mang project mới lên AWS đi khoe khắp nơi.*
