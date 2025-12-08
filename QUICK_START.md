# Quick Start Guide

Get up and running with GitHub Workflows Library in 5 minutes.

## 📋 Prerequisites

Before you begin, ensure you have:

1. ✅ AWS account with S3 bucket created
2. ✅ IAM role configured with OIDC federation
3. ✅ GitHub repository with Actions enabled
4. ✅ Application that builds to static files

## 🚀 Step 1: Set Up AWS

### Create S3 Bucket

```bash
aws s3 mb s3://my-app-bucket --region us-east-1
```

### Create IAM Role for OIDC

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:*"
        }
      }
    }
  ]
}
```

### Attach Policy to Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::my-app-bucket",
        "arn:aws:s3:::my-app-bucket/*"
      ]
    }
  ]
}
```

## 🔧 Step 2: Configure GitHub

### Add Repository Variables

Go to Settings → Secrets and variables → Actions → Variables:

```
AWS_REGION = us-east-1
AWS_ACCOUNT_ID_PRODUCTION = 123456789012
S3_BUCKET_PRODUCTION = my-app-bucket
```

### Add Repository Secrets (Optional)

For notifications:

```
SLACK_WEBHOOK_URL = https://hooks.slack.com/services/...
```

## 📝 Step 3: Create Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy App

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1
    with:
      environment: production
      aws_region: ${{ vars.AWS_REGION }}
      aws_role_arn: arn:aws:iam::${{ vars.AWS_ACCOUNT_ID_PRODUCTION }}:role/github-actions-role
      s3_bucket: ${{ vars.S3_BUCKET_PRODUCTION }}
      build_command: npm run build
      build_output_dir: dist
    permissions:
      id-token: write
      contents: read
```

## 🎉 Step 4: Deploy

Push to main branch:

```bash
git add .github/workflows/deploy.yml
git commit -m "feat: add deployment workflow"
git push origin main
```

Watch your deployment in the Actions tab!

## 🔄 Step 5: Add Rollback (Optional)

Create `.github/workflows/rollback.yml`:

```yaml
name: Rollback App

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [production]

jobs:
  rollback:
    uses: carlssonk/cicd-toolkit/.github/workflows/rollback-s3.yml@v1
    with:
      environment: ${{ inputs.environment }}
      aws_region: ${{ vars.AWS_REGION }}
      aws_role_arn: arn:aws:iam::${{ vars.AWS_ACCOUNT_ID_PRODUCTION }}:role/github-actions-role
      s3_bucket: ${{ vars.S3_BUCKET_PRODUCTION }}
    permissions:
      id-token: write
      contents: read
```

## 📊 Step 6: Add Notifications (Optional)

Update your deploy workflow:

```yaml
jobs:
  deploy:
    # ... existing deploy job

  notify:
    needs: deploy
    if: always()
    uses: carlssonk/cicd-toolkit/.github/workflows/notify.yml@v1
    with:
      notification_type: slack
      status: ${{ needs.deploy.result }}
      title: "Production Deployment"
      environment: production
      commit_hash: ${{ needs.deploy.outputs.commit_hash }}
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## ✅ Verification

After deployment, verify:

1. Check S3 bucket has two paths:
   - `s3://my-app-bucket/{commit-hash}/`
   - `s3://my-app-bucket/latest/`

2. Check metadata:
```bash
aws s3api head-object \
  --bucket my-app-bucket \
  --key latest/index.html \
  --query Metadata
```

3. Test your application at the S3 URL

## 🎯 Next Steps

- [Set up multi-environment deployment](./docs/examples/multi-env-deploy.yml)
- [Add release tagging](./docs/create-release.md)

## 🆘 Troubleshooting

### "Access Denied" Error

- Verify IAM role ARN is correct
- Check OIDC trust policy
- Ensure role has S3 permissions

### "Build Output Not Found"

- Verify `build_output_dir` matches your build tool
- Check build command succeeded

### Workflow Not Running

- Check workflow file is in `.github/workflows/`
- Verify YAML syntax is correct
- Check branch name matches trigger

## 📚 Learn More

- [Full Documentation](./README.md)
- [All Workflows](./README.md#-available-workflows)
- [Examples](./docs/examples/)
- [Best Practices](./README.md#-best-practices)

---

Need help? [Open a discussion](https://github.com/carlssonk/cicd-toolkit/discussions)

