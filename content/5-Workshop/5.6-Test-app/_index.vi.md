---
title: "Kết nối app Flutter & Kiểm thử"
date: 2026-06-10
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

Đến lúc gặt thành quả: trỏ app Flutter thật vào AWS, chạy trên điện thoại Android thật, và xem request đổ về CloudWatch.

### Bước 1 — Migration một dòng

Vì API AWS là drop-in replacement, `lib/services/api_config.dart` chỉ cần thêm một hằng số, và các getter URL của phim + auth chuyển sang dùng nó:

```dart
class ApiConfig {
  // Backend serverless AWS (API Gateway + Lambda + DynamoDB)
  static const String awsBaseUrl =
      'https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod';

  static String getMoviesLimitUrl(int limit) =>
      '$awsBaseUrl$movieEndpoint/limit/$limit';
  static String get loginUrl => '$awsBaseUrl$authEndpoint/login';
  // ... đổi tương tự cho các getter phim/auth còn lại
}
```

Không sửa màn hình, provider, model hay widget nào — format JSON giống hệt nhau.

### Bước 2 — Build lên máy thật (và các chướng ngại)

```bash
flutter run -d <device-id> --release
```

Hai lỗi build có thật và cách sửa — giữ lại để tự debug:

{{% notice warning %}}
**Lỗi 1: `File google-services.json is missing`** — file config Firebase nằm trong gitignore, nên clone mới không có. Cách sửa: Firebase Console → Project settings → app Android của bạn → tải `google-services.json` → bỏ vào `android/app/`.
{{% /notice %}}

{{% notice warning %}}
**Lỗi 2: Kotlin `Daemon compilation failed ... this and base files have different roots`** — xảy ra khi project (ổ `E:`) và pub cache (ổ `C:`) nằm khác ổ đĩa. Cách sửa: thêm `kotlin.incremental=false` vào `android/gradle.properties` và xóa thư mục `build/`.
{{% /notice %}}

📸 *Screenshot: màn hình chính của app đầy phim — phục vụ bởi Lambda.*

### Bước 3 — Test app end-to-end

Trên điện thoại (server Node cũ TẮT hoàn toàn):

- ✅ Trang chủ load danh mục phim
- ✅ Lọc thể loại, tìm kiếm, chi tiết phim, **phát video** (m3u8 từ DynamoDB)
- ✅ Đăng ký → OTP về email → xác minh → đăng nhập
- ✅ Trường hợp lỗi: sai mật khẩu → báo lỗi; tài khoản chưa xác minh → bị chặn

### Bước 4 — Xem trực tiếp trong CloudWatch

Console → **CloudWatch → Log groups** → `/aws/lambda/cinemax-movies`:

```
INFO  Request: GET /api/movies/limit/20
REPORT ... Duration: 389 ms  Billed Duration: 390 ms  Memory Size: 256 MB
```

Mỗi cú chạm trong app trở thành một dòng log. Dòng `REPORT` là đồng hồ đo hiệu năng + chi phí. 📸

Hoặc theo dõi trực tiếp từ terminal trong lúc dùng app:

```bash
sam logs --stack-name cinemax-serverless --tail
```

### Bước 5 — Chứng minh alarm hoạt động

1. Lambda → `cinemax-movies` → Configuration → Environment variables → đổi `MOVIES_TABLE` thành `wrong-table`.
2. Refresh app vài lần → mọi API call trả về 500.
3. CloudWatch → Alarms → `cinemax-api-5xx-errors` chuyển **In alarm** 🔴 trong ~5 phút. 📸
4. Khôi phục biến → app hồi phục → alarm về **OK** ✅.

### Kết quả

App giờ hoạt động **ở bất cứ đâu có internet** — tắt máy tính, không VPS, không tiến trình MongoDB. Đường đi đầy đủ của request: điện thoại Android → API Gateway (Singapore) → Lambda → DynamoDB → quay về, giám sát end-to-end bằng CloudWatch.
