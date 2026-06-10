---
title: "Chuẩn bị"
date: 2026-06-10
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

Trước khi bắt đầu, hãy đảm bảo bạn có đầy đủ những thứ sau.

### 1. Tài khoản AWS

- Một [tài khoản AWS Free Tier](https://aws.amazon.com/free/).
- **Region:** workshop này dùng `ap-southeast-1` (Singapore). Region nào cũng được, nhưng phải dùng nhất quán xuyên suốt các bước.

{{% notice warning %}}
Hãy thiết lập **AWS Budget alert** (ngưỡng $1) trước khi làm bất cứ điều gì khác: Console → Billing → Budgets → Create budget. Nếu lỡ quên tắt tài nguyên nào, bạn sẽ nhận email cảnh báo trước khi tốn tiền thật.
{{% /notice %}}

### 2. IAM user (không dùng tài khoản root)

1. Mở **IAM** trong console → **Users** → **Create user** (ví dụ `cinemax-dev`).
2. Gắn quyền cần thiết cho workshop (với mục đích học tập, `AdministratorAccess` chấp nhận được trên tài khoản sandbox cá nhân; trong môi trường production cần giới hạn quyền chặt hơn).
3. Tạo **Access key** (Use case: CLI) và lưu giữ an toàn.

{{% notice info %}}
Tuyệt đối không commit access key lên GitHub. File `.gitignore` của project đã loại trừ các file chứa credential, và code Lambda không bao giờ chứa key — khi chạy, nó dùng IAM Role do SAM gắn vào.
{{% /notice %}}

### 3. Công cụ cài trên máy

| Công cụ | Lệnh kiểm tra | Hướng dẫn cài |
|---|---|---|
| AWS CLI v2 | `aws --version` | [Cài AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) |
| AWS SAM CLI | `sam --version` | [Cài SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html) |
| Node.js ≥ 20 | `node --version` | [nodejs.org](https://nodejs.org) |
| Git | `git --version` | [git-scm.com](https://git-scm.com) |

Cấu hình CLI với access key của IAM user:

```bash
aws configure
# AWS Access Key ID: <key của bạn>
# AWS Secret Access Key: <secret của bạn>
# Default region name: ap-southeast-1
# Default output format: json
```

Kiểm tra hoạt động:

```bash
aws sts get-caller-identity
```

Bạn sẽ thấy account ID và ARN của user `cinemax-dev`.

### 4. Mã nguồn project

```bash
git clone https://github.com/nguyenwoong20/cinemax-serverless-aws.git
cd cinemax-serverless-aws
npm install
```

Repository gồm:

```
cinemax-serverless-aws/
├── template.yaml          # SAM template — toàn bộ tài nguyên AWS
├── src/handlers/
│   ├── movies.js          # Lambda: CRUD + tìm kiếm phim
│   └── bookmarks.js       # Lambda: bookmark người dùng
├── scripts/seed.js        # nhập dữ liệu phim từ MongoDB export vào DynamoDB
└── package.json
```

Bạn cũng cần file `appxemphim.movies.json` — dữ liệu phim xuất từ MongoDB gốc (có trong repository [cinemax-backend](https://github.com/nguyenwoong20/cinemax-backend)).
