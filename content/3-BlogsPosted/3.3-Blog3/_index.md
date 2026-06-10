---
title: "Blog 3: Monitoring & Cost Control on the Free Tier"
date: 2026-06-10
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

{{% notice note %}}
✏️ Draft — post to the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) and paste the post link here.
{{% /notice %}}

**Posted at:** [LINK TO FACEBOOK POST]

# Sleep Well on the Free Tier: Monitoring and Cost Control for a Student AWS Project

The two fears every student has about AWS: "what if it breaks and I don't know" and "what if I get a surprise bill". Both are solvable in under 30 minutes. Here's what I set up for my serverless movie API.

**1. A $1 Budget alert — before anything else.** Billing → Budgets → Create budget → $1 threshold. This is the single highest-value action in this post. I created it before deploying anything, so any mistake emails me long before it costs real money.

**2. CloudWatch Logs are free observability.** Every `console.log` in a Lambda lands in CloudWatch automatically. The `REPORT` line at the end of each invocation tells you duration and memory — that's your performance profiler and your cost calculator in one line. Tip: `sam logs --stack-name <stack> --tail` streams them live to your terminal.

**3. One metric alarm catches what logs can't.** Logs only help if you look at them. I added a CloudWatch Alarm on API Gateway's `5XXError` metric (≥ 5 errors in 5 minutes). I tested it by intentionally breaking my Lambda's environment variable — within minutes the alarm went red. Fixed the variable, it went green. Now failures announce themselves.

**4. Architecture choices ARE cost controls.** On-demand DynamoDB instead of provisioned capacity, arm64 Lambdas (20% cheaper), S3 instead of an always-on file server — my whole stack runs at ≈ $0.02/month. The cheapest resource is the one that only exists while it's working.

**5. Clean-up is part of the project.** Because everything lives in one SAM stack, `sam delete` removes all of it. If you can't delete your project in one command, you don't fully know what you're paying for.

#AWS #CloudWatch #FreeTier #CostOptimization #FirstCloudJourney
