---
title: "Week 9 Worklog"
date: 2026-06-10
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

{{% notice note %}}
✏️ Draft — adjust to match what you actually did this week (add dates, details, reference links).
{{% /notice %}}

### Week 9 Objectives

- Rebuild the entire authentication system serverless.

### Week 9 Achievements

- Implemented the cinemax-auth Lambda with all 7 endpoints of the old backend: register, verify-email (OTP), login, google-login, resend OTP, forgot/reset password.
- Sent real OTP emails from inside Lambda using nodemailer + Gmail app password; secrets passed as NoEcho CloudFormation parameters.
- Verified Google idToken server-side with google-auth-library; kept the same JWT secret so existing tokens remain valid.
- Created the cinemax-users table (PK: email) with bcrypt-hashed passwords; tested the full register → OTP → verify → login flow with curl.
