---
title: "Blog 1: From Express Server to Serverless"
date: 2026-06-10
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

{{% notice note %}}
✏️ Draft — post to the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) and paste the post link here.
{{% /notice %}}

**Posted at:** [LINK TO FACEBOOK POST]

# From an Always-On Express Server to Serverless: What I Learned Migrating My Movie App Backend

My movie app Cinemax used to run on a Node.js/Express server with MongoDB. It worked — but the server billed me 24/7 even when nobody opened the app. Migrating to AWS serverless taught me a few things worth sharing.

**1. You don't rewrite everything — you re-map it.** An Express route like `router.get('/movies/:id')` maps almost 1:1 to an API Gateway resource + a Lambda handler. The business logic barely changes; what changes is *where* it runs.

**2. Pay-per-request changes how you think.** With Lambda's free tier (1M invocations/month) and DynamoDB on-demand, my entire backend now costs about $0.02/month. The mental shift: stop asking "how big a server do I need" and start asking "how much work does each request do".

**3. The connection problem disappears.** MongoDB + Lambda is famously awkward (connection pooling across cold starts). DynamoDB's HTTP-based API has no connections at all — every Lambda invocation just signs a request.

**4. IAM roles replace credentials in code.** My old backend had a MongoDB connection string in an env file. The Lambda version has *zero* secrets: SAM attaches an IAM role that allows exactly one table per function. If one function is compromised, the blast radius is one table.

If you have an Express + MongoDB side project, re-architecting even part of it into Lambda + DynamoDB is one of the fastest ways to actually understand serverless instead of just reading about it.

#AWS #Serverless #Lambda #DynamoDB #FirstCloudJourney
