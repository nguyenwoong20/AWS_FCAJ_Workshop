---
title: "Dọn dẹp tài nguyên"
date: 2026-06-10
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

Để tránh phát sinh chi phí về sau, hãy xóa toàn bộ tài nguyên đã tạo trong workshop. Vì mọi thứ được tạo bởi một stack SAM/CloudFormation duy nhất, việc dọn dẹp gần như chỉ cần một lệnh.

### Bước 1 — Làm trống bucket S3

CloudFormation không thể xóa bucket còn chứa object:

```bash
aws s3 rm s3://cinemax-posters-<account-id> --recursive
```

### Bước 2 — Xóa stack

```bash
sam delete --stack-name cinemax-serverless
# Are you sure you want to delete the stack? [y/N]: y
# Are you sure you want to delete the folder ... in S3? [y/N]: y
```

Lệnh này xóa lần lượt: API Gateway, cả hai Lambda function cùng IAM role của chúng, cả hai bảng DynamoDB (bao gồm toàn bộ dữ liệu), bucket poster, và CloudWatch alarm.

📸 *Screenshot: thông báo `Deleted successfully`.*

### Bước 3 — Kiểm tra không còn gì sót lại

| Trang console | Kết quả mong đợi |
|---|---|
| CloudFormation → Stacks | `cinemax-serverless` biến mất (hoặc `DELETE_COMPLETE`) |
| Lambda → Functions | không còn function `cinemax-*` |
| DynamoDB → Tables | không còn bảng `cinemax-*` |
| S3 → Buckets | không còn bucket `cinemax-posters-*` |
| CloudWatch → Alarms | không còn `cinemax-api-5xx-errors` |

📸 *Screenshot: các danh sách trống.*

{{% notice tip %}}
**Log groups** của CloudWatch (`/aws/lambda/cinemax-*`) có thể vẫn còn — kích thước này không tốn tiền, nhưng bạn cũng có thể xóa: CloudWatch → Log groups → chọn → Actions → Delete.
{{% /notice %}}

### Bước 4 — Kiểm tra hóa đơn lần cuối

Console → **Billing** → **Bills**: xác nhận chi phí trong tháng là $0 (hoặc chỉ vài cent cho dung lượng S3 dùng trong workshop). Giữ cảnh báo ngân sách $1 hoạt động như một lưới an toàn.

🎉 **Workshop hoàn thành.** Bạn đã xây dựng, kiểm thử, giám sát và dọn dẹp sạch sẽ một backend serverless thực thụ trên AWS.
