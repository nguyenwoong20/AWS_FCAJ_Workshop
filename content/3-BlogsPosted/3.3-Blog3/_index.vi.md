---
title: "Blog 3: Giám sát & kiểm soát chi phí trong Free Tier"
date: 2026-06-10
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

{{% notice note %}}
✏️ Bản nháp — đăng lên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) rồi dán link bài đăng vào đây.
{{% /notice %}}

**Đã đăng tại:** [LINK BÀI ĐĂNG FACEBOOK]

# Ngủ ngon với Free Tier: Giám sát và kiểm soát chi phí cho dự án AWS sinh viên

Hai nỗi sợ của mọi sinh viên khi dùng AWS: "lỡ nó hỏng mà mình không biết" và "lỡ nhận hóa đơn bất ngờ". Cả hai đều giải quyết được trong chưa đầy 30 phút. Đây là những gì tôi thiết lập cho API phim serverless của mình.

**1. Budget alert $1 — trước mọi thứ khác.** Billing → Budgets → Create budget → ngưỡng $1. Đây là hành động giá trị nhất trong cả bài viết này. Tôi tạo nó trước khi deploy bất cứ thứ gì, nên mọi sai sót sẽ gửi email cho tôi từ rất lâu trước khi tốn tiền thật.

**2. CloudWatch Logs là observability miễn phí.** Mọi `console.log` trong Lambda tự động đổ vào CloudWatch. Dòng `REPORT` cuối mỗi lần chạy cho biết thời gian và bộ nhớ — vừa là profiler hiệu năng vừa là máy tính chi phí trong một dòng. Mẹo: `sam logs --stack-name <stack> --tail` để xem log trực tiếp trong terminal.

**3. Một metric alarm bắt được thứ mà log không bắt được.** Log chỉ hữu ích nếu bạn nhìn vào nó. Tôi thêm CloudWatch Alarm trên metric `5XXError` của API Gateway (≥ 5 lỗi trong 5 phút). Tôi test bằng cách cố ý làm sai biến môi trường của Lambda — vài phút sau alarm chuyển đỏ. Sửa lại biến, nó chuyển xanh. Giờ đây sự cố tự thông báo về mình.

**4. Lựa chọn kiến trúc CHÍNH LÀ kiểm soát chi phí.** DynamoDB on-demand thay vì provisioned, Lambda arm64 (rẻ hơn 20%), S3 thay vì file server chạy liên tục — cả stack của tôi tốn ≈ $0.02/tháng. Tài nguyên rẻ nhất là tài nguyên chỉ tồn tại khi nó đang làm việc.

**5. Clean-up là một phần của dự án.** Vì mọi thứ nằm trong một SAM stack, `sam delete` xóa được tất cả. Nếu bạn không thể xóa dự án của mình bằng một lệnh, nghĩa là bạn chưa thực sự biết mình đang trả tiền cho cái gì.

#AWS #CloudWatch #FreeTier #CostOptimization #FirstCloudJourney
