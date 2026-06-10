---
title: "Workshop"
date: 2026-06-10
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Migrating a Real Flutter App Backend to AWS Serverless

In this workshop, you will migrate the backend of **Cinemax** — a real movie streaming Flutter app — from a traditional Node.js/Express + MongoDB server to a **fully serverless architecture on AWS**, using a *drop-in replacement* strategy: the new cloud API mimics the old API's routes and response shapes exactly, so the mobile app needs only a one-line base URL change.

By the end, the app's movie catalog, search, playback metadata **and the entire authentication system** (register, email OTP, login, Google Sign-In, password reset) run on AWS with no server to maintain and ≈ $0 monthly cost.

![Cinemax Serverless Architecture](/images/2-Proposal/architecture.svg)

### Content

1. [Workshop Overview](5.1-workshop-overview/)
2. [Prerequisites](5.2-prerequiste/)
3. [Deploy the Serverless Backend](5.3-deploy-backend/)
4. [Data Migration from MongoDB](5.4-data-migration/)
5. [Serverless Authentication](5.5-auth-api/)
6. [Connect the Flutter App & Test](5.6-test-app/)
7. [Clean-up](5.7-cleanup/)
