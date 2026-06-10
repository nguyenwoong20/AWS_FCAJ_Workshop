---
title: "Proposal"
date: 2026-06-10
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Cinemax Serverless API
## Re-architecting a Movie Streaming App Backend with AWS Serverless

### 1. Executive Summary

Cinemax is a movie streaming mobile application built with Flutter that I developed previously. Its current backend runs on Node.js/Express with MongoDB and must be hosted on an always-on server, which costs money even when no one is using the app and cannot scale automatically when traffic spikes.

This project re-architects the core of the Cinemax backend (movie catalog and user bookmarks) into a **fully serverless architecture on AWS**, using Amazon API Gateway, AWS Lambda, Amazon DynamoDB, Amazon S3 and Amazon CloudWatch. The result is a backend that costs almost nothing at low traffic, scales automatically, and requires zero server maintenance.

### 2. Problem Statement

#### What's the problem?

- The current Express + MongoDB backend requires a VPS or hosting service that runs **24/7**, even when the app has no users.
- Scaling requires manual work (bigger server, load balancer setup).
- There is no built-in monitoring or alerting; when the API fails, nobody knows until users complain.
- Movie poster images are served from third-party URLs, which can break at any time.

#### The solution

Move the core API to AWS serverless services:

- **Amazon API Gateway** exposes the REST API.
- **AWS Lambda** (Node.js 22) runs the business logic only when a request arrives — pay per invocation.
- **Amazon DynamoDB** (on-demand mode) stores movies and bookmarks — pay per request.
- **Amazon S3** hosts movie poster images reliably.
- **Amazon CloudWatch** collects logs from every Lambda invocation and raises an alarm when the API returns 5XX errors.

### 3. Solution Architecture

![Cinemax Serverless Architecture](/images/2-Proposal/architecture.svg)

#### AWS Services Used (5)

| Service | Role |
|---|---|
| Amazon API Gateway | REST API endpoint for the Flutter app (`/api/movies/*`, `/api/auth/*`, bookmarks) |
| AWS Lambda | Three functions: `cinemax-movies` (catalog + search), `cinemax-auth` (register/login/OTP/Google) and `cinemax-bookmarks` |
| Amazon DynamoDB | Three tables: `cinemax-movies` (GSIs: `slug-index`, `category-createdAt-index`), `cinemax-users` (bcrypt-hashed passwords) and `cinemax-bookmarks` |
| Amazon S3 | Poster image storage, public read restricted to the `posters/` prefix only |
| Amazon CloudWatch | Lambda logs, API metrics, 5XX error alarm |

#### Why these services?

- **Lambda + API Gateway**: no idle cost, automatic scaling, and I already know JavaScript/Node.js from the original backend.
- **DynamoDB on-demand**: free tier covers 25 GB and millions of requests; no connection pooling problems like MongoDB on Lambda.
- **S3**: 99.999999999% durability for images, far more reliable than hot-linking third-party URLs.
- **CloudWatch**: built-in, no agent to install.
- **AWS SAM** is used as Infrastructure-as-Code so the whole stack can be deployed or deleted with one command.

### 4. Technical Implementation

1. **Design** the DynamoDB data model (movies table + GSI, bookmarks table with composite key `userId` + `movieId`).
2. **Develop** Lambda handlers in Node.js using AWS SDK v3.
3. **Define** all resources in a SAM template (`template.yaml`) with least-privilege IAM policies per function.
4. **Deploy** with `sam build && sam deploy`.
5. **Migrate data**: a seed script imports the real movie data exported from the original MongoDB database into DynamoDB.
6. **Test**: call every endpoint with curl/Postman, verify logs in CloudWatch, trigger the error alarm intentionally.
7. **Connect** the Flutter app to the new API base URL.

### 5. Timeline & Milestones

- **Weeks 1–3**: Learn AWS fundamentals (IAM, Lambda, DynamoDB, S3, CloudWatch).
- **Weeks 4–5**: Architecture design + this proposal.
- **Weeks 6–9**: Implementation: SAM template, Lambda code, data migration, S3 posters.
- **Weeks 10–11**: Testing, monitoring setup, documentation (this workshop).
- **Week 12**: Final report, clean-up, presentation.

### 6. Budget Estimation

With AWS Free Tier (first 12 months):

| Service | Usage estimate | Monthly cost |
|---|---|---|
| Lambda | < 10,000 invocations | $0.00 (free tier: 1M/month) |
| API Gateway | < 10,000 requests | $0.00 (free tier: 1M/month, first 12 months) |
| DynamoDB | < 1 GB, on-demand | $0.00 (free tier: 25 GB) |
| S3 | ~500 MB posters | $0.02 |
| CloudWatch | logs + 1 alarm | $0.00 (free tier: 10 alarms) |

**Total: ≈ $0.02/month** — effectively free during the internship.

### 7. Risk Assessment

| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| Unexpected AWS charges | Medium | Low | AWS Budget alert at $1; on-demand billing; clean-up guide |
| DynamoDB data model mistakes | Medium | Medium | Design GSI up front; test queries before migrating all data |
| Lambda cold start latency | Low | High | Acceptable for this use case (< 1s with arm64 + small bundle) |
| Free Tier expiry | Low | Low | Project completes within the free tier period |

### 8. Expected Outcomes

- A working serverless API serving the real Cinemax movie dataset.
- The Flutter app running against the AWS backend.
- A reproducible step-by-step workshop so anyone can deploy the same stack.
- Monitoring with CloudWatch logs and an automatic 5XX alarm.
- Total running cost ≈ $0 and zero servers to maintain.
