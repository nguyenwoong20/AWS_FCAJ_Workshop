---
title: "Blog 1: Từ server Express sang Serverless"
date: 2026-06-10
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

{{% notice note %}}
✏️ Bản nháp — đăng lên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) rồi dán link bài đăng vào đây.
{{% /notice %}}

**Đã đăng tại:** [LINK BÀI ĐĂNG FACEBOOK]

# Từ server Express chạy 24/7 sang Serverless: Những điều tôi học được khi chuyển backend app xem phim

App xem phim Cinemax của tôi từng chạy trên server Node.js/Express với MongoDB. Nó hoạt động tốt — nhưng server tính tiền 24/7 kể cả khi không ai mở app. Việc chuyển sang AWS serverless dạy tôi vài điều đáng chia sẻ.

**1. Bạn không viết lại tất cả — bạn ánh xạ lại.** Một route Express như `router.get('/movies/:id')` ánh xạ gần như 1:1 sang một resource API Gateway + một Lambda handler. Logic nghiệp vụ hầu như không đổi; thứ thay đổi là *nơi* nó chạy.

**2. Trả tiền theo request thay đổi cách bạn tư duy.** Với free tier của Lambda (1 triệu lần gọi/tháng) và DynamoDB on-demand, toàn bộ backend của tôi giờ tốn khoảng $0.02/tháng. Thay đổi tư duy: đừng hỏi "cần server to cỡ nào" mà hãy hỏi "mỗi request tốn bao nhiêu công việc".

**3. Vấn đề connection biến mất.** MongoDB + Lambda nổi tiếng rắc rối (connection pooling qua các lần cold start). API của DynamoDB chạy trên HTTP, không có connection nào cả — mỗi lần Lambda chạy chỉ cần ký một request.

**4. IAM role thay thế credential trong code.** Backend cũ của tôi có connection string MongoDB trong file env. Phiên bản Lambda có *zero* secret: SAM gắn IAM role cho phép đúng một bảng cho mỗi function. Nếu một function bị tấn công, phạm vi thiệt hại chỉ là một bảng.

Nếu bạn có side project Express + MongoDB, tái kiến trúc dù chỉ một phần sang Lambda + DynamoDB là cách nhanh nhất để thực sự hiểu serverless thay vì chỉ đọc về nó.

#AWS #Serverless #Lambda #DynamoDB #FirstCloudJourney
