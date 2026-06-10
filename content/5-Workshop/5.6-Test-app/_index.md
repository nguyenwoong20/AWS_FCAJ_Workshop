---
title: "Connect the Flutter App & Test"
date: 2026-06-10
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

Time for the payoff: pointing the real Flutter app at AWS, running it on a physical Android phone, and watching the requests in CloudWatch.

### Step 1 — The one-line migration

Because the AWS API is a drop-in replacement, `lib/services/api_config.dart` only needs a new constant, and the movie + auth URL getters switch to it:

```dart
class ApiConfig {
  // AWS serverless backend (API Gateway + Lambda + DynamoDB)
  static const String awsBaseUrl =
      'https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod';

  static String getMoviesLimitUrl(int limit) =>
      '$awsBaseUrl$movieEndpoint/limit/$limit';
  static String get loginUrl => '$awsBaseUrl$authEndpoint/login';
  // ... same change for the other movie/auth getters
}
```

No screen, provider, model or widget changes — the JSON shapes are identical.

### Step 2 — Build onto a real device (and the obstacles)

```bash
flutter run -d <device-id> --release
```

Two real build failures and their fixes — keep these for your own debugging:

{{% notice warning %}}
**Failure 1: `File google-services.json is missing`** — the Firebase config file is gitignored, so a fresh clone doesn't have it. Fix: Firebase Console → Project settings → your Android app → download `google-services.json` → put it in `android/app/`.
{{% /notice %}}

{{% notice warning %}}
**Failure 2: Kotlin `Daemon compilation failed ... this and base files have different roots`** — happens when the project (drive `E:`) and the pub cache (drive `C:`) live on different drives. Fix: add `kotlin.incremental=false` to `android/gradle.properties` and delete the `build/` folder.
{{% /notice %}}

📸 *Screenshot: the app's home screen full of movies — served by Lambda.*

### Step 3 — Test the app end-to-end

On the phone (old Node server completely OFF):

- ✅ Home screen loads the movie catalog
- ✅ Category filter, search, movie detail, **video playback** (m3u8 from DynamoDB)
- ✅ Register → OTP email arrives → verify → login
- ✅ Error cases: wrong password → message; unverified account → blocked

### Step 4 — Watch it live in CloudWatch

Console → **CloudWatch → Log groups** → `/aws/lambda/cinemax-movies`:

```
INFO  Request: GET /api/movies/limit/20
REPORT ... Duration: 389 ms  Billed Duration: 390 ms  Memory Size: 256 MB
```

Every tap in the app becomes a log line. The `REPORT` line is your performance + cost meter. 📸

Or stream live from the terminal while using the app:

```bash
sam logs --stack-name cinemax-serverless --tail
```

### Step 5 — Prove the alarm works

1. Lambda → `cinemax-movies` → Configuration → Environment variables → change `MOVIES_TABLE` to `wrong-table`.
2. Refresh the app a few times → every API call returns 500.
3. CloudWatch → Alarms → `cinemax-api-5xx-errors` turns **In alarm** 🔴 within ~5 minutes. 📸
4. Restore the variable → app recovers → alarm back to **OK** ✅.

### Result

The app now works **anywhere with internet** — PC off, no VPS, no MongoDB process. The complete request path: Android phone → API Gateway (Singapore) → Lambda → DynamoDB → back, observed end-to-end in CloudWatch.
