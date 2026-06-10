---
title: "Week 8 Worklog"
date: 2026-06-10
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

{{% notice note %}}
✏️ Draft — adjust to match what you actually did this week (add dates, details, reference links).
{{% /notice %}}

### Week 8 Objectives

- Migrate the real movie data from MongoDB and survive the 400 KB item limit.

### Week 8 Achievements

- Exported 240 movies from MongoDB (mongoexport) and wrote scripts/seed.js importing in batches of 25 (BatchWriteItem limit).
- Hit a real ValidationException: long TV series exceeded DynamoDB's 400 KB item limit — MongoDB never complained at 16 MB.
- Fixed it with progressive compaction: drop non-essential episode fields first, then keep only the first streaming server; playback stays intact.
- Verified slug-index and category queries from the CLI against the seeded data.
