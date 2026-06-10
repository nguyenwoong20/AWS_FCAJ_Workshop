---
title: "Deploy the Serverless Backend"
date: 2026-06-10
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

In this step we deploy the entire infrastructure — API Gateway, three Lambda functions, three DynamoDB tables, the S3 bucket and the CloudWatch alarm — with AWS SAM.

### Step 1 — Understand the SAM template

Open `template.yaml`. The movies table is designed around the app's real access patterns:

```yaml
MoviesTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: cinemax-movies
    BillingMode: PAY_PER_REQUEST        # on-demand: pay only per request
    KeySchema:
      - AttributeName: id
        KeyType: HASH                   # get by id
    GlobalSecondaryIndexes:
      - IndexName: slug-index           # app fetches movie detail BY SLUG
        KeySchema:
          - AttributeName: slug
            KeyType: HASH
      - IndexName: category-createdAt-index   # newest movies in a category
        KeySchema:
          - AttributeName: category
            KeyType: HASH
          - AttributeName: createdAt
            KeyType: RANGE
```

{{% notice info %}}
**Why `slug-index`?** The Flutter app calls `GET /api/movies/{slug}` (e.g. `/api/movies/366-ngay`) for the detail screen — that's how the old Express API worked. A GSI on `slug` lets us answer that with a fast Query instead of a full table Scan. Design follows the *client's* access patterns, not the data.
{{% /notice %}}

Routing matches the old Express API exactly — one proxy route per function:

```yaml
MoviesFunction:
  Events:
    MovieRoutes:
      Type: Api
      Properties:
        Path: /api/movies/{proxy+}    # limit/20, category/hanh-dong, {slug}...
        Method: ANY
```

Each Lambda gets a **least-privilege policy** scoped to its own table only:

```yaml
Policies:
  - DynamoDBCrudPolicy:
      TableName: !Ref MoviesTable   # this table, nothing else
```

Secrets (JWT secret, email password, Google client IDs) are **CloudFormation parameters** with `NoEcho: true` — they are never committed to Git.

### Step 2 — Build

```bash
sam build
```

Expected output: `Build Succeeded`. 📸

### Step 3 — Deploy

```bash
sam deploy --guided
```

| Prompt | Answer |
|---|---|
| Stack Name | `cinemax-serverless` |
| AWS Region | `ap-southeast-1` |
| Parameter JwtSecret | *(your JWT secret — same as the old backend so existing tokens stay valid)* |
| Parameter EmailUser / EmailPass | *(Gmail + app password for OTP emails)* |
| Parameter GoogleClientId / GoogleAndroidClientId | *(from Google Cloud Console)* |
| Allow SAM CLI IAM role creation | `y` |
| Functions have no authorization defined, Is this okay? | `y` |

After 2–3 minutes:

```
Successfully created/updated stack - cinemax-serverless
```

Copy the **ApiUrl** output: `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod` 📸

### Step 4 — Verify in the console

1. **CloudFormation** → stack `cinemax-serverless` → `CREATE_COMPLETE` 📸
2. **Lambda** → `cinemax-movies`, `cinemax-auth`, `cinemax-bookmarks` 📸
3. **DynamoDB** → 3 tables; open `cinemax-movies` → tab *Indexes* → 2 GSIs 📸
4. **API Gateway** → `cinemax-api` → routes `/api/movies/{proxy+}`, `/api/auth/{proxy+}` 📸

### Step 5 — Smoke test

```bash
curl https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod/api/movies/limit/5
```

Expected (table still empty):

```json
{"success":true,"data":[]}
```

The backend skeleton is alive. Next: fill it with the real movie catalog.
