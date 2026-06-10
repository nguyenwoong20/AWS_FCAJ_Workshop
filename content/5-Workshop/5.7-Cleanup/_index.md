---
title: "Clean-up"
date: 2026-06-10
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

To avoid any future charges, remove every resource created in this workshop. Because everything was created by one SAM/CloudFormation stack, clean-up is almost a single command.

### Step 1 — Empty the S3 bucket

CloudFormation cannot delete a bucket that still contains objects:

```bash
aws s3 rm s3://cinemax-posters-<account-id> --recursive
```

### Step 2 — Delete the stack

```bash
sam delete --stack-name cinemax-serverless
# Are you sure you want to delete the stack? [y/N]: y
# Are you sure you want to delete the folder ... in S3? [y/N]: y
```

This removes, in order: the API Gateway, both Lambda functions and their IAM roles, both DynamoDB tables (including all data), the posters bucket, and the CloudWatch alarm.

📸 *Screenshot: `Deleted successfully` message.*

### Step 3 — Verify nothing is left

| Console page | Expected |
|---|---|
| CloudFormation → Stacks | `cinemax-serverless` gone (or `DELETE_COMPLETE`) |
| Lambda → Functions | no `cinemax-*` functions |
| DynamoDB → Tables | no `cinemax-*` tables |
| S3 → Buckets | no `cinemax-posters-*` bucket |
| CloudWatch → Alarms | no `cinemax-api-5xx-errors` |

📸 *Screenshot: empty lists.*

{{% notice tip %}}
CloudWatch **Log groups** (`/aws/lambda/cinemax-*`) may remain — they cost nothing at this size, but you can delete them too: CloudWatch → Log groups → select → Actions → Delete.
{{% /notice %}}

### Step 4 — Final billing check

Console → **Billing** → **Bills**: confirm the month's charges are $0 (or only a few cents for S3 storage used during the workshop). Keep the $1 budget alert active as a safety net.

🎉 **The workshop is complete.** You built, tested, monitored, and cleanly removed a real serverless backend on AWS.
