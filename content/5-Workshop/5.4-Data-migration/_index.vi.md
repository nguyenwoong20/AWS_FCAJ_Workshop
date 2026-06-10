---
title: "Di chuyển dữ liệu từ MongoDB"
date: 2026-06-10
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Backend cũ lưu 240 phim trong MongoDB. Chúng ta export và import vào DynamoDB — và đụng một giới hạn thực tế giữa đường.

### Bước 1 — Export từ MongoDB

```bash
mongoexport --db appxemphim --collection movies --jsonArray --out appxemphim.movies.json
```

Mỗi document theo format phimapi.com: `name`, `slug`, `content`, `category` (mảng `{name, slug}`), `episodes` (server → danh sách tập với link phát `link_m3u8`)...

### Bước 2 — Thiết kế item DynamoDB

Script seed (`scripts/seed.js`) lưu mỗi phim thành **hai lớp trong một item**:

```javascript
{
  // trường nhẹ top-level → endpoint danh sách dùng với ProjectionExpression
  id, slug, name, originName, posterUrl, thumbUrl, year,
  episodeCurrent, quality, lang,
  categoryNames: ['Chính Kịch', 'Tình Cảm'],
  categorySlugs: 'chinh-kich,tinh-cam',   // để lọc bằng contains()
  createdAt,
  // document gốc ĐẦY ĐỦ → endpoint chi tiết trả về nguyên văn
  doc: { ...tất cả, episodes: [...] }
}
```

Nhờ vậy màn hình danh sách không bao giờ tốn tiền đọc mảng `episodes` nặng nề, còn màn hình chi tiết nhận đúng JSON mà app vốn đã biết parse.

### Bước 3 — Chạy import (và đụng tường 400 KB)

```bash
node scripts/seed.js appxemphim.movies.json
```

Lần chạy đầu — lỗi thật ở item ~60:

```
ValidationException: Item size has exceeded the maximum allowed size
```

{{% notice warning %}}
**Bài học thực tế:** item DynamoDB tối đa **400 KB**. Phim bộ dài tập mang theo hàng trăm tập trên nhiều server — một số document vượt giới hạn. MongoDB không bao giờ phàn nàn (giới hạn của nó là 16 MB), nên vấn đề chỉ lộ ra khi migration.
{{% /notice %}}

**Cách sửa** — nén lũy tiến trong script seed, vẫn giữ phát phim hoạt động:

```javascript
function compact(doc) {
  if (size(doc) <= 350000) return doc;
  // 1) bỏ các trường tập phim cồng kềnh không thiết yếu
  for (const server of doc.episodes || [])
    for (const ep of server.server_data || []) {
      delete ep.filename;
      delete ep.link_embed;   // app phát bằng link_m3u8
    }
  if (size(doc) <= 350000) return doc;
  // 2) chỉ giữ server phát đầu tiên
  doc.episodes = [doc.episodes[0]];
  return doc;
}
```

Chạy lần hai:

```
Seeding 240 movies into cinemax-movies...
  wrote 25/240 ... wrote 240/240
Done.
```

📸 *Screenshot: output terminal + DynamoDB console → Explore table items.*

### Bước 4 — Kiểm tra index hoạt động

```bash
# chi tiết theo slug (slug-index)
curl "$API/api/movies/366-ngay"
# → document đầy đủ kèm episodes + link_m3u8

# phim mới nhất theo thể loại
curl "$API/api/movies/category/hanh-dong?limit=3"
# → {"success":true,"data":[ 3 phim hành động ]}
```

### Bước 5 — Poster lên S3 (tăng cường, tùy chọn)

URL poster hiện trỏ về host bên thứ ba. Để tự chủ, upload vào bucket của stack — bucket policy chỉ mở public read cho prefix `posters/`:

```bash
aws s3 cp ./posters/ s3://cinemax-posters-<account-id>/posters/ --recursive
curl -I https://cinemax-posters-<account-id>.s3.ap-southeast-1.amazonaws.com/posters/movie1.jpg   # 200 OK
```

Tầng dữ liệu hoàn tất: 240 phim thật trong DynamoDB, truy vấn được đúng theo cách app yêu cầu.
