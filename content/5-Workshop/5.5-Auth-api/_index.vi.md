---
title: "Xác thực Serverless"
date: 2026-06-10
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Backend cũ xử lý đăng ký/đăng nhập bằng Express + MongoDB + nodemailer. Chúng ta xây lại cả 7 endpoint auth trong một Lambda duy nhất — cùng route, cùng format response, cùng JWT secret, nên token tương thích hoàn toàn.

### Các endpoint

| Route (`POST /api/auth/...`) | Chức năng |
|---|---|
| `register` | Validate → băm mật khẩu bcrypt → lưu user → gửi OTP 6 số qua email |
| `verify-email` | Kiểm tra OTP + hạn 5 phút → đánh dấu tài khoản đã xác minh |
| `login` | So sánh bcrypt → yêu cầu tài khoản đã xác minh → ký JWT (7 ngày) |
| `google-login` | Xác minh Google `idToken` phía server → upsert user → ký JWT |
| `resend-verify-otp` | Rate-limit: 60 giây giữa các lần gửi, tối đa 3 lần |
| `forgot-password` | Gửi OTP đặt lại mật khẩu |
| `reset-password` | Kiểm tra OTP reset → băm bcrypt mật khẩu mới |

### Bước 1 — Bảng users

User khóa theo email — mọi thao tác auth chỉ là một lệnh `GetItem`:

```yaml
UsersTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: cinemax-users
    BillingMode: PAY_PER_REQUEST
    KeySchema:
      - AttributeName: email
        KeyType: HASH
```

### Bước 2 — Bảo mật mật khẩu

Mật khẩu không bao giờ chạm database ở dạng thô:

```javascript
// đăng ký
password: await bcrypt.hash(password, 10),

// đăng nhập
if (!(await bcrypt.compare(password, user.password)))
  return response(400, { success: false, message: 'Invalid email or password' });
```

### Bước 3 — Gửi OTP email ngay trong Lambda

Lambda dùng được nodemailer + Gmail **app password** y như server Express — không cần thiết lập SES ở quy mô này:

```javascript
const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: { user: process.env.EMAIL_USER, pass: process.env.EMAIL_PASS },
});
```

Credential đi vào qua CloudFormation parameter (`NoEcho`) → biến môi trường Lambda. Không hard-code gì cả.

{{% notice tip %}}
Ở quy mô production bạn sẽ chuyển sang **Amazon SES** (rẻ hơn, không bị giới hạn của Gmail). Với app cá nhân, ~500 email/ngày của Gmail là quá đủ — một đánh đổi chi phí/độ đơn giản có chủ đích, đáng nêu trong phần thiết kế.
{{% /notice %}}

### Bước 4 — Xác minh Google Sign-In

App Flutter lấy `idToken` qua Firebase/Google Sign-In, và Lambda xác minh **phía server** — không bao giờ tin client:

```javascript
const ticket = await googleClient.verifyIdToken({
  idToken: googleToken,
  audience: [process.env.GOOGLE_CLIENT_ID, process.env.GOOGLE_ANDROID_CLIENT_ID],
});
const { email, name, sub: googleId, picture } = ticket.getPayload();
```

### Bước 5 — Test trọn luồng

```bash
API=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod

# 1. Đăng ký → kiểm tra hộp thư lấy OTP
curl -X POST "$API/api/auth/register" -H "Content-Type: application/json" \
  -d '{"name":"Test","email":"you@gmail.com","password":"test123"}'
# {"success":true,"message":"OTP has been sent to your email"}

# 2. Đăng nhập trước khi xác minh → bị chặn đúng logic
curl -X POST "$API/api/auth/login" -H "Content-Type: application/json" \
  -d '{"email":"you@gmail.com","password":"test123"}'
# {"success":false,"message":"Please verify your email"}

# 3. Xác minh bằng OTP trong email
curl -X POST "$API/api/auth/verify-email" -H "Content-Type: application/json" \
  -d '{"email":"you@gmail.com","otp":"123456"}'
# {"success":true,"message":"Email verified successfully"}

# 4. Đăng nhập lại → JWT + thông tin user
curl -X POST "$API/api/auth/login" -H "Content-Type: application/json" \
  -d '{"email":"you@gmail.com","password":"test123"}'
# {"success":true,"token":"eyJhbGciOi...","auth":{"id":"...","name":"Test",...}}
```

📸 *Screenshot: email OTP trong hộp thư + từng response curl + item user trong DynamoDB console.*

Toàn bộ hệ thống đăng nhập giờ chạy không cần server nào — tiếp theo, trỏ app thật vào nó.
