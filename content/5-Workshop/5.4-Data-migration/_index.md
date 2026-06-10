---
title: "Data Migration from MongoDB"
date: 2026-06-10
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

The old backend stored 240 movies in MongoDB. We export them and import into DynamoDB — and hit a real-world limit along the way.

### Step 1 — Export from MongoDB

```bash
mongoexport --db appxemphim --collection movies --jsonArray --out appxemphim.movies.json
```

Each document follows the phimapi.com format: `name`, `slug`, `content`, `category` (array of `{name, slug}`), `episodes` (servers → episode list with `link_m3u8` playback URLs)...

### Step 2 — Design the DynamoDB item

The seed script (`scripts/seed.js`) stores each movie as **two layers in one item**:

```javascript
{
  // small top-level fields → used by list endpoints with ProjectionExpression
  id, slug, name, originName, posterUrl, thumbUrl, year,
  episodeCurrent, quality, lang,
  categoryNames: ['Chính Kịch', 'Tình Cảm'],
  categorySlugs: 'chinh-kich,tinh-cam',   // for contains() filtering
  createdAt,
  // the FULL original document → returned as-is by the detail endpoint
  doc: { ...everything, episodes: [...] }
}
```

This way list screens never pay to read the heavy `episodes` array, while the detail screen gets the exact JSON the app already knows how to parse.

### Step 3 — Run the import (and hit the 400 KB wall)

```bash
node scripts/seed.js appxemphim.movies.json
```

First attempt — a real error at item ~60:

```
ValidationException: Item size has exceeded the maximum allowed size
```

{{% notice warning %}}
**Real-world lesson:** DynamoDB items max out at **400 KB**. Long-running TV series carry hundreds of episodes across multiple servers — some documents exceeded the limit. MongoDB never complained (its limit is 16 MB), so this only surfaces during migration.
{{% /notice %}}

**The fix** — progressive compaction in the seed script, keeping playback intact:

```javascript
function compact(doc) {
  if (size(doc) <= 350000) return doc;
  // 1) drop bulky non-essential episode fields
  for (const server of doc.episodes || [])
    for (const ep of server.server_data || []) {
      delete ep.filename;
      delete ep.link_embed;   // app plays link_m3u8
    }
  if (size(doc) <= 350000) return doc;
  // 2) keep only the first streaming server
  doc.episodes = [doc.episodes[0]];
  return doc;
}
```

Second run:

```
Seeding 240 movies into cinemax-movies...
  wrote 25/240 ... wrote 240/240
Done.
```

📸 *Screenshot: terminal output + DynamoDB console → Explore table items.*

### Step 4 — Verify the indexes work

```bash
# detail by slug (slug-index)
curl "$API/api/movies/366-ngay"
# → full document with episodes + link_m3u8

# newest in a category
curl "$API/api/movies/category/hanh-dong?limit=3"
# → {"success":true,"data":[ 3 action movies ]}
```

### Step 5 — Posters on S3 (optional hardening)

Poster URLs currently point at a third-party host. To own them, upload to the stack's bucket — public read is locked to the `posters/` prefix only by the bucket policy:

```bash
aws s3 cp ./posters/ s3://cinemax-posters-<account-id>/posters/ --recursive
curl -I https://cinemax-posters-<account-id>.s3.ap-southeast-1.amazonaws.com/posters/movie1.jpg   # 200 OK
```

The data layer is complete: 240 real movies in DynamoDB, queryable exactly the way the app asks for them.
