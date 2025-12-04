# Deploy to S3 with Versioning

A reusable GitHub Actions workflow for deploying applications to AWS S3 with built-in versioning and rollback support.

## Features

- ✅ **Versioned Deployments**: Each deployment is stored with its commit hash for easy rollback
- ✅ **Dual-Path Strategy**: Immutable versioned path + mutable main path
- ✅ **Metadata Tracking**: Tracks deployment timestamps, commit hashes, and previous versions
- ✅ **Flexible Build System**: Supports npm, yarn, and pnpm
- ✅ **Concurrency Control**: Prevents race conditions during deployment
- ✅ **OIDC Authentication**: Secure AWS authentication without long-lived credentials

## Usage

### Basic Example

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

## Inputs

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `environment` | Environment name | `production` |
| `aws_region` | AWS region | `us-east-1` |
| `aws_role_arn` | AWS IAM role ARN for OIDC | `arn:aws:iam::123456789012:role/...` |
| `s3_bucket` | S3 bucket name | `my-app-bucket` |
| `build_command` | Command to build the app | `npm run build` |
| `build_output_dir` | Build output directory | `dist` or `build` |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `node_version` | Node.js version | `20` |
| `package_manager` | Package manager (npm/yarn/pnpm) | `npm` |
| `package_manager_version` | Package manager version (pnpm only) | `""` |
| `install_command` | Custom install command | Auto-detected |
| `cache_control_immutable` | Cache-Control for versioned files | `public, max-age=31536000, immutable` |
| `cache_control_mutable` | Cache-Control for main path | `public, max-age=300` |
| `commit_hash_length` | Length of commit hash | `8` |
| `s3_additional_args` | Additional aws s3 sync args | `""` |

## Outputs

| Output | Description |
|--------|-------------|
| `commit_hash` | The commit hash that was deployed |
| `previous_commit_hash` | The previous commit hash |
| `deployment_timestamp` | Timestamp of deployment |
| `s3_versioned_url` | S3 URL for versioned deployment |
| `s3_main_url` | S3 URL for main deployment |

## How It Works

### Deployment Strategy

The workflow uses a dual-path deployment strategy:

1. **Versioned Path** (`s3://bucket/{commit-hash}/`)
   - Immutable deployment
   - Long cache duration (1 year)
   - Never deleted
   - Used for rollbacks

2. **Main Path** (`s3://bucket/main/`)
   - Mutable deployment
   - Short cache duration (5 minutes)
   - Updated with each deployment
   - Tracks previous version in metadata

### Metadata

Each deployment includes metadata:
- `deployed-at`: Timestamp
- `commit-hash-short`: Current commit
- `prev-commit-hash-short`: Previous commit (for rollback)
- `environment`: Environment name

### Rollback Support

The workflow automatically tracks the previous deployment, enabling one-click rollbacks using the companion `rollback-s3.yml` workflow.

## Prerequisites

### AWS Setup

1. **S3 Bucket**: Create an S3 bucket for your application
2. **IAM Role**: Create an IAM role with OIDC federation
3. **Permissions**: Ensure the role has:
   - `s3:PutObject`
   - `s3:GetObject`
   - `s3:ListBucket`
   - `s3:DeleteObject`

### GitHub Setup

1. Configure OIDC federation with AWS
2. Set up repository variables:
   - `AWS_REGION`
   - Environment-specific variables as needed

## Examples

See the [examples](./examples/) directory for complete working examples:
- [React App Deployment](./examples/react-app-deploy.yml)
- [Vue App Deployment](./examples/vue-app-deploy.yml)
- [Multi-Environment Deployment](./examples/multi-env-deploy.yml)

## Best Practices

1. **Use OIDC**: Never use long-lived AWS credentials
2. **Pin Versions**: Use specific versions or commit SHAs when referencing this workflow
3. **Set Timeouts**: The workflow includes a 30-minute timeout by default
5. **Test First**: Deploy to staging before production

## Troubleshooting

### Build Output Directory Not Found

Ensure your `build_output_dir` matches your build tool's output:
- Create React App: `build`
- Vite: `dist`
- Next.js: `out` (with `output: 'export'`)

### AWS Permissions Issues

Verify your IAM role has the required permissions and trust policy for OIDC.

## Related Workflows

- [Rollback S3 Deployment](./rollback-s3.md)
- [Create Release](./create-release.md)

