---
title: "Triển khai Backend Serverless"
date: 2026-06-10
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

Trong bước này, chúng ta triển khai toàn bộ hạ tầng — API Gateway, ba Lambda function, ba bảng DynamoDB, bucket S3 và CloudWatch alarm — bằng AWS SAM.

### Bước 1 — Hiểu SAM template

Mở `template.yaml`. Bảng movies được thiết kế theo đúng access pattern thật của app:

```yaml
MoviesTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: cinemax-movies
    BillingMode: PAY_PER_REQUEST        # on-demand: chỉ trả tiền theo request
    KeySchema:
      - AttributeName: id
        KeyType: HASH                   # lấy theo id
    GlobalSecondaryIndexes:
      - IndexName: slug-index           # app lấy chi tiết phim THEO SLUG
        KeySchema:
          - AttributeName: slug
            KeyType: HASH
      - IndexName: category-createdAt-index   # phim mới nhất theo thể loại
        KeySchema:
          - AttributeName: category
            KeyType: HASH
          - AttributeName: createdAt
            KeyType: RANGE
```

{{% notice info %}}
**Vì sao có `slug-index`?** App Flutter gọi `GET /api/movies/{slug}` (ví dụ `/api/movies/366-ngay`) cho màn hình chi tiết — đó là cách API Express cũ hoạt động. GSI trên `slug` cho phép trả lời bằng Query nhanh thay vì Scan cả bảng. Thiết kế đi theo access pattern của *client*, không phải theo dữ liệu.
{{% /notice %}}

Định tuyến khớp chính xác API Express cũ — mỗi function một route proxy:

```yaml
MoviesFunction:
  Events:
    MovieRoutes:
      Type: Api
      Properties:
        Path: /api/movies/{proxy+}    # limit/20, category/hanh-dong, {slug}...
        Method: ANY
```

Mỗi Lambda có **policy tối thiểu** chỉ trong phạm vi bảng của nó:

```yaml
Policies:
  - DynamoDBCrudPolicy:
      TableName: !Ref MoviesTable   # chỉ bảng này, không gì khác
```

Các secret (JWT secret, mật khẩu email, Google client ID) là **CloudFormation parameter** với `NoEcho: true` — không bao giờ commit lên Git.

### Bước 2 — Build

```bash
sam build
```

Kết quả mong đợi: `Build Succeeded`. 📸

### Bước 3 — Deploy

```bash
sam deploy --guided
```

| Câu hỏi | Trả lời |
|---|---|
| Stack Name | `cinemax-serverless` |
| AWS Region | `ap-southeast-1` |
| Parameter JwtSecret | *(JWT secret của bạn — giữ giống backend cũ để token hiện có vẫn hợp lệ)* |
| Parameter EmailUser / EmailPass | *(Gmail + app password để gửi OTP)* |
| Parameter GoogleClientId / GoogleAndroidClientId | *(từ Google Cloud Console)* |
| Allow SAM CLI IAM role creation | `y` |
| Functions have no authorization defined, Is this okay? | `y` |

Sau 2–3 phút:

```
Successfully created/updated stack - cinemax-serverless
```

Sao chép output **ApiUrl**: `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod` 📸

### Bước 4 — Kiểm tra trong console

1. **CloudFormation** → stack `cinemax-serverless` → `CREATE_COMPLETE` 📸
2. **Lambda** → `cinemax-movies`, `cinemax-auth`, `cinemax-bookmarks` 📸
3. **DynamoDB** → 3 bảng; mở `cinemax-movies` → tab *Indexes* → 2 GSI 📸
4. **API Gateway** → `cinemax-api` → route `/api/movies/{proxy+}`, `/api/auth/{proxy+}` 📸

### Bước 5 — Smoke test

```bash
curl https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod/api/movies/limit/5
```

Kết quả mong đợi (bảng còn trống):

```json
{"success":true,"data":[]}
```

Khung backend đã sống. Tiếp theo: nạp danh mục phim thật vào.
