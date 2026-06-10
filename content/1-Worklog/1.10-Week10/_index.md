---
title: "Week 10 Worklog"
date: 2026-06-10
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

{{% notice note %}}
✏️ Draft — adjust to match what you actually did this week (add dates, details, reference links).
{{% /notice %}}

### Week 10 Objectives

- Connect the real Flutter app, test end-to-end on a physical device, verify monitoring.

### Week 10 Achievements

- Migrated the app by changing one constant in api_config.dart — movie + auth endpoints now point at API Gateway.
- Solved two real build failures: missing google-services.json (gitignored Firebase config) and the Kotlin incremental-compiler bug when project and pub cache sit on different drives (kotlin.incremental=false).
- Ran the app on a physical Android phone with the old server OFF: browse, search, playback, register/OTP/login all served by AWS.
- Watched live requests in CloudWatch Logs, measured Duration/Memory from REPORT lines, and intentionally triggered + restored the 5XX alarm.
