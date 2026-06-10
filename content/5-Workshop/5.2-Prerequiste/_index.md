---
title: "Prerequisites"
date: 2026-06-10
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

Before starting, make sure you have all of the following.

### 1. AWS account

- An [AWS Free Tier account](https://aws.amazon.com/free/).
- **Region:** this workshop uses `ap-southeast-1` (Singapore). Any region works, but keep it consistent through every step.

{{% notice warning %}}
Set up an **AWS Budget alert** ($1 threshold) before doing anything else: Console → Billing → Budgets → Create budget. If anything is accidentally left running, you will be emailed before it costs real money.
{{% /notice %}}

### 2. IAM user (do not use the root account)

1. Open **IAM** in the console → **Users** → **Create user** (e.g. `cinemax-dev`).
2. Attach the permissions needed for this workshop (for learning purposes `AdministratorAccess` is acceptable on a personal sandbox account; in production you would scope this down).
3. Create an **Access key** (Use case: CLI) and save it securely.

{{% notice info %}}
Never commit access keys to GitHub. The project's `.gitignore` already excludes credential files, and the Lambda code never contains a key — at runtime it uses the IAM Role attached by SAM.
{{% /notice %}}

### 3. Local tools

| Tool | Check command | Install guide |
|---|---|---|
| AWS CLI v2 | `aws --version` | [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) |
| AWS SAM CLI | `sam --version` | [Install SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html) |
| Node.js ≥ 20 | `node --version` | [nodejs.org](https://nodejs.org) |
| Git | `git --version` | [git-scm.com](https://git-scm.com) |

Configure the CLI with your IAM user's access key:

```bash
aws configure
# AWS Access Key ID: <your key>
# AWS Secret Access Key: <your secret>
# Default region name: ap-southeast-1
# Default output format: json
```

Verify it works:

```bash
aws sts get-caller-identity
```

You should see your account ID and the `cinemax-dev` user ARN.

### 4. Project source code

```bash
git clone https://github.com/nguyenwoong20/cinemax-serverless-aws.git
cd cinemax-serverless-aws
npm install
```

The repository contains:

```
cinemax-serverless-aws/
├── template.yaml          # SAM template — all AWS resources
├── src/handlers/
│   ├── movies.js          # Lambda: movie CRUD + search
│   └── bookmarks.js       # Lambda: user bookmarks
├── scripts/seed.js        # imports MongoDB movie export into DynamoDB
└── package.json
```

You also need `appxemphim.movies.json` — the movie data exported from the original MongoDB database (available in the [cinemax-backend](https://github.com/nguyenwoong20/cinemax-backend) repository).
