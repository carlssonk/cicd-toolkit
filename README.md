# GitHub Workflows Library

A collection of production-ready, reusable GitHub Actions workflows for modern application deployment and release management.

## 🚀 Features

- **S3 Deployment with Versioning**: Deploy applications to S3 with automatic versioning and rollback support
- **Intelligent Rollback**: One-click rollback to previous versions or specific commits
- **Release Management**: Automated release tagging with changelog generation
- **Deployment Routing**: Smart routing based on branching strategy (trunk-based, GitFlow, GitHub Flow)
- **Multi-Platform Notifications**: Send notifications to Slack, Discord, Teams, or GitHub Discussions
- **Production-Ready**: Built with best practices, security, and reliability in mind

## 📦 Available Workflows

| Workflow | Description | Documentation |
|----------|-------------|---------------|
| **deploy-s3.yml** | Deploy applications to S3 with versioning and CloudFront support | [Docs](./docs/deploy-s3.md) |
| **rollback-s3.yml** | Rollback S3 deployments to previous versions | [Docs](./docs/rollback-s3.md) |
| **create-release.yml** | Create release tags and GitHub releases with changelogs | [Docs](./docs/create-release.md) |
| **notify.yml** | Send notifications to various platforms | [Docs](./docs/notify.md) |

## 🎯 Quick Start

### 1. Basic S3 Deployment

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
      aws_region: us-east-1
      aws_role_arn: arn:aws:iam::123456789012:role/github-actions-role
      s3_bucket: my-app-bucket
      build_command: npm run build
      build_output_dir: dist
    permissions:
      id-token: write
      contents: read
```

### 2. Complete Deployment Pipeline

```yaml
name: Deploy and Release

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1
    with:
      environment: production
      aws_region: us-east-1
      aws_role_arn: ${{ secrets.AWS_ROLE_ARN }}
      s3_bucket: my-app-bucket
      build_command: npm run build
      build_output_dir: dist
      enable_cloudfront_invalidation: true
      cloudfront_distribution_id: E1ABCDEFGHIJK
    permissions:
      id-token: write
      contents: read

  create-release:
    needs: deploy
    if: needs.deploy.result == 'success'
    uses: carlssonk/cicd-toolkit/.github/workflows/create-release.yml@v1
    with:
      environment: production
      commit_hash: ${{ needs.deploy.outputs.commit_hash }}
      generate_changelog: true
    permissions:
      contents: write

  notify:
    needs: [deploy, create-release]
    if: always()
    uses: carlssonk/cicd-toolkit/.github/workflows/notify.yml@v1
    with:
      notification_type: slack
      status: ${{ needs.deploy.result }}
      title: "Production Deployment"
      environment: production
      commit_hash: ${{ needs.deploy.outputs.commit_hash }}
      deployment_url: ${{ needs.create-release.outputs.release_url }}
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### 3. Rollback Workflow

```yaml
name: Rollback App

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [staging, production]

jobs:
  rollback:
    uses: carlssonk/cicd-toolkit/.github/workflows/rollback-s3.yml@v1
    with:
      environment: ${{ inputs.environment }}
      aws_region: us-east-1
      aws_role_arn: ${{ secrets.AWS_ROLE_ARN }}
      s3_bucket: my-app-bucket
    permissions:
      id-token: write
      contents: read
```

## 📚 Documentation

### Workflow Documentation

- [Deploy to S3](./docs/deploy-s3.md) - Comprehensive S3 deployment with versioning
- [Rollback S3](./docs/rollback-s3.md) - Rollback deployments safely
- [Create Release](./docs/create-release.md) - Automated release management
- [Send Notification](./docs/notify.md) - Multi-platform notifications

### Examples

- [React App Deployment](./docs/examples/react-app-deploy.yml) - Complete React app deployment
- [Rollback Example](./docs/examples/rollback-example.yml) - Rollback with notifications
- [Multi-Environment Deployment](./docs/examples/multi-env-deploy.yml) - Multi-region deployment

## 🏗️ Architecture

### S3 Deployment Strategy

The workflows use a dual-path deployment strategy:

```
s3://bucket/
├── {commit-hash}/          # Versioned, immutable deployments
│   ├── index.html
│   ├── assets/
│   └── ...
└── main/                   # Current deployment (mutable)
    ├── index.html
    ├── assets/
    └── ...
```

**Benefits:**
- ✅ Instant rollbacks (no rebuild required)
- ✅ Immutable version history
- ✅ Optimal caching strategy
- ✅ Metadata tracking for audit trail

### Metadata Tracking

Each deployment includes metadata:
- `commit-hash-short`: Current commit
- `prev-commit-hash-short`: Previous commit (for rollback)
- `deployed-at`: Deployment timestamp
- `environment`: Environment name
- `rolled-back-from`: Source commit (for rollbacks)
- `rollback-by`: User who performed rollback

## 🔒 Security Best Practices

### 1. OIDC Authentication

All workflows use OIDC for AWS authentication - no long-lived credentials:

```yaml
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789012:role/github-actions-role
    aws-region: us-east-1
```

### 2. Minimal Permissions

Workflows request only the permissions they need:

```yaml
permissions:
  id-token: write    # For OIDC
  contents: read     # For checkout
  # contents: write  # Only when creating releases
```

### 3. Concurrency Control

Prevents race conditions during deployment:

```yaml
concurrency:
  group: s3-deploy-${{ inputs.environment }}-${{ inputs.s3_bucket }}
  cancel-in-progress: false
```

## 🛠️ Prerequisites

### AWS Setup

1. **S3 Bucket**: Create an S3 bucket for your application
2. **IAM Role**: Create an IAM role with OIDC federation
3. **Permissions**: Grant the role:
   - `s3:PutObject`, `s3:GetObject`, `s3:ListBucket`, `s3:DeleteObject`
   - `cloudfront:CreateInvalidation` (if using CloudFront)

### GitHub Setup

1. **OIDC Configuration**: Configure OIDC federation with AWS
2. **Repository Variables**: Set up environment-specific variables
3. **Secrets**: Store webhook URLs and sensitive data

## 🎨 Supported Frameworks

These workflows support any framework that builds to static files:

- ✅ React (Create React App, Vite)
- ✅ Vue (Vue CLI, Vite)
- ✅ Angular
- ✅ Svelte
- ✅ Next.js (with `output: 'export'`)
- ✅ Gatsby
- ✅ Hugo
- ✅ Any static site generator

## 📊 Workflow Features

### Deploy to S3

- Versioned deployments with commit hashes
- Automatic metadata tracking
- CloudFront cache invalidation
- Support for npm, yarn, and pnpm
- Concurrency control
- Build verification
- Comprehensive output variables

### Rollback S3

- Automatic rollback to previous version
- Manual rollback to specific commit
- Validation before rollback
- Metadata updates
- CloudFront invalidation support
- Safety checks

### Create Release

- Automatic git tagging
- GitHub release creation
- Changelog generation from conventional commits
- Customizable tag prefixes
- Pre-release support
- Idempotent operation

### Deployment Router

- Multiple branching strategies
- Event-aware routing
- Custom routing with JSON
- Environment determination
- Action determination (plan/deploy)

### Notify

- Multi-platform support (Slack, Discord, Teams, GitHub Discussions)
- Status-based formatting
- Rich context (commit info, links, etc.)
- Mention on failure
- Customizable messages

## 🚦 Best Practices

### 1. Version Pinning

Always pin workflow versions:

```yaml
# ✅ Good - pinned to specific version
uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1.2.3

# ⚠️ Acceptable - pinned to major version
uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1

# ❌ Bad - uses latest, breaking changes possible
uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@main
```

### 2. Environment Strategy

Use GitHub Environments for production:

```yaml
jobs:
  deploy-production:
    environment:
      name: production
      url: https://app.example.com
    uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1
    # ...
```

### 3. Secrets Management

Store sensitive data in GitHub secrets:

```yaml
with:
  aws_role_arn: arn:aws:iam::${{ secrets.AWS_ACCOUNT_ID }}:role/github-actions-role
  s3_bucket: ${{ vars.S3_BUCKET_PRODUCTION }}
secrets:
  SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### 4. Testing Pipeline

Always test in lower environments first:

```
dev → staging → production
```

### 5. Monitoring

- Set up notifications for deployment status
- Monitor CloudWatch metrics
- Track deployment frequency and success rate

## 🐛 Troubleshooting

### Common Issues

1. **AWS Permission Denied**
   - Verify IAM role has required permissions
   - Check OIDC trust policy
   - Ensure role ARN is correct

2. **Build Output Not Found**
   - Verify `build_output_dir` matches your build tool
   - Check build command succeeded
   - Ensure build output isn't gitignored

3. **CloudFront Invalidation Fails**
   - Verify distribution ID is correct
   - Check IAM permissions include `cloudfront:CreateInvalidation`
   - Ensure distribution is in "Deployed" state

4. **Rollback: "No Previous Version"**
   - This is the first deployment, or
   - Deployment was made before metadata tracking
   - Use manual rollback with specific commit hash

## 📈 Metrics & Monitoring

Track these key metrics:

- **Deployment Frequency**: How often you deploy
- **Lead Time**: Time from commit to production
- **Change Failure Rate**: % of deployments causing issues
- **Mean Time to Recovery (MTTR)**: Time to rollback/fix

## 🤝 Contributing

Contributions are welcome! Please:

1. Follow existing code style
2. Add tests for new features
3. Update documentation
4. Submit a pull request

## 📄 License

[MIT License](./LICENSE)

## 🙏 Acknowledgments

Built with best practices from:
- GitHub Actions documentation
- AWS deployment patterns
- DevOps community standards

## 📞 Support

- 📖 [Documentation](./docs/)
- 💬 [GitHub Discussions](https://github.com/carlssonk/cicd-toolkit/discussions)
- 🐛 [Issue Tracker](https://github.com/carlssonk/cicd-toolkit/issues)

---

**Made ~~with ❤️ for the DevOps community~~ by a Clanker**

