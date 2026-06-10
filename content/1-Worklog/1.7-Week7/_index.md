---
title: "Week 7 Worklog"
date: 2026-06-10
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

{{% notice note %}}
✏️ Draft — adjust to match what you actually did this week (add dates, details, reference links).
{{% /notice %}}

### Week 7 Objectives

- Implement the movies Lambda as a drop-in replacement for the old Express API.

### Week 7 Achievements

- Analyzed how the Flutter app calls the old API (routes, response shapes, detail-by-slug) and chose the drop-in replacement strategy.
- Wrote the cinemax-movies handler: /api/movies/limit, /category/{slug}, /country/{slug}, /year, /{slug} detail, search — all returning {success, data} like Express did.
- Added the slug-index GSI so the detail screen resolves by slug with a Query instead of a Scan.
- Designed two-layer items: light top-level fields for lists (ProjectionExpression), full original document in a `doc` attribute for detail.
