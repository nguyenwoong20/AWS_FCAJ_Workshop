---
title: "Serverless Authentication"
date: 2026-06-10
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

The old backend handled register/login with Express + MongoDB + nodemailer. We rebuild all 7 auth endpoints in a single Lambda — same routes, same response shapes, same JWT secret, so tokens stay compatible.

### The endpoints

| Route (`POST /api/auth/...`) | What it does |
|---|---|
| `register` | Validate → bcrypt-hash the password → save user → email a 6-digit OTP |
| `verify-email` | Check OTP + 5-minute expiry → mark account verified |
| `login` | bcrypt compare → require verified account → sign JWT (7 days) |
| `google-login` | Verify Google `idToken` server-side → upsert user → sign JWT |
| `resend-verify-otp` | Rate-limited: 60 s between sends, max 3 resends |
| `forgot-password` | Email a reset OTP |
| `reset-password` | Check reset OTP → bcrypt-hash the new password |

### Step 1 — The users table

Users are keyed by email — every auth operation is a single `GetItem`:

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

### Step 2 — Password security

Passwords never touch the database in plain text:

```javascript
// register
password: await bcrypt.hash(password, 10),

// login
if (!(await bcrypt.compare(password, user.password)))
  return response(400, { success: false, message: 'Invalid email or password' });
```

### Step 3 — OTP emails from inside Lambda

A Lambda function can use nodemailer + a Gmail **app password** exactly like an Express server — no SES setup needed for this scale:

```javascript
const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: { user: process.env.EMAIL_USER, pass: process.env.EMAIL_PASS },
});
```

The credentials arrive as CloudFormation parameters (`NoEcho`) → Lambda environment variables. Nothing is hard-coded.

{{% notice tip %}}
At production scale you would switch to **Amazon SES** (cheaper, no Gmail rate limits). For a personal app, Gmail's ~500 emails/day is plenty — a deliberate cost/simplicity trade-off worth mentioning in your design review.
{{% /notice %}}

### Step 4 — Google Sign-In verification

The Flutter app obtains an `idToken` via Firebase/Google Sign-In, and Lambda verifies it **server-side** — never trust the client:

```javascript
const ticket = await googleClient.verifyIdToken({
  idToken: googleToken,
  audience: [process.env.GOOGLE_CLIENT_ID, process.env.GOOGLE_ANDROID_CLIENT_ID],
});
const { email, name, sub: googleId, picture } = ticket.getPayload();
```

### Step 5 — Test the full flow

```bash
API=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod

# 1. Register → check your inbox for the OTP
curl -X POST "$API/api/auth/register" -H "Content-Type: application/json" \
  -d '{"name":"Test","email":"you@gmail.com","password":"test123"}'
# {"success":true,"message":"OTP has been sent to your email"}

# 2. Login before verifying → correctly rejected
curl -X POST "$API/api/auth/login" -H "Content-Type: application/json" \
  -d '{"email":"you@gmail.com","password":"test123"}'
# {"success":false,"message":"Please verify your email"}

# 3. Verify with the OTP from the email
curl -X POST "$API/api/auth/verify-email" -H "Content-Type: application/json" \
  -d '{"email":"you@gmail.com","otp":"123456"}'
# {"success":true,"message":"Email verified successfully"}

# 4. Login again → JWT + user profile
curl -X POST "$API/api/auth/login" -H "Content-Type: application/json" \
  -d '{"email":"you@gmail.com","password":"test123"}'
# {"success":true,"token":"eyJhbGciOi...","auth":{"id":"...","name":"Test",...}}
```

📸 *Screenshots: the OTP email in your inbox + each curl response + the user item in DynamoDB console.*

The entire login system now runs without any server — next, point the real app at it.
