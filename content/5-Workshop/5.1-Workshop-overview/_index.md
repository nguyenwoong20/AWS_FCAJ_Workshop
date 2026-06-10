---
title: "Workshop Overview"
date: 2026-06-10
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### Background & problem

Cinemax is a movie streaming app I built with Flutter. Its backend was a Node.js/Express server with MongoDB that I had to run on my own machine (or rent a VPS for):

- The server must run **24/7** even with zero users — wasted cost.
- The app only works when my PC/server is online.
- No monitoring: when the API died, nobody knew.
- Scaling would require manual server upgrades.

**Goal:** move the backend to AWS serverless so the app works anywhere, anytime, scales automatically, costs ≈ $0 at student traffic levels — **without rewriting the mobile app**.

### The key strategy: drop-in replacement

Instead of redesigning the API (which would force changes across the whole Flutter codebase), the new serverless backend **mimics the old Express API exactly** — same routes (`/api/movies/limit/{n}`, `/api/movies/category/{slug}`, `/api/auth/login`...), same JSON response shapes (`{success: true, data: [...]}`), same JWT secret.

Result: migrating the app = changing **one constant** in `api_config.dart`.

### What you will build

| Component | AWS resource |
|---|---|
| Movie API (list, detail by slug, search, category/country/year filters) | Lambda `cinemax-movies` + DynamoDB `cinemax-movies` (GSI: `slug-index`, `category-createdAt-index`) |
| Full authentication (register, email OTP, login, Google Sign-In, password reset) | Lambda `cinemax-auth` + DynamoDB `cinemax-users` + Gmail SMTP |
| User bookmarks | Lambda `cinemax-bookmarks` + DynamoDB `cinemax-bookmarks` |
| REST routing | Amazon API Gateway (`/api/movies/*`, `/api/auth/*`) |
| Poster storage | Amazon S3 (public read on `posters/` prefix only) |
| Observability | CloudWatch Logs + 5XX error alarm |

Everything is defined in **one SAM template** (`template.yaml`) — Infrastructure-as-Code, deployable and removable with one command.

### What you will learn

- Re-architecting a real Express + MongoDB backend into Lambda + DynamoDB.
- DynamoDB modeling from real access patterns (and surviving the **400 KB item limit** with real data).
- Running email OTP and Google token verification inside Lambda.
- Least-privilege IAM: each function can only touch its own table; zero hard-coded credentials.
- Reading CloudWatch logs, testing an alarm, and connecting a real Android device end-to-end.

### Duration & cost

- **Duration:** 2–3 hours end-to-end.
- **Cost:** ≈ $0 with AWS Free Tier (see [Proposal](../../2-proposal/)).
