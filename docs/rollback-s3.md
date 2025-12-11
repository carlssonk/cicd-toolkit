# Rollback S3 Deployment

A reusable GitHub Actions workflow for rolling back S3 deployments to a previous version.

## Features

- ✅ **Automatic Rollback**: Roll back to the previous deployment automatically
- ✅ **Manual Rollback**: Specify any commit hash to roll back to
- ✅ **Validation**: Verifies target commit exists before rolling back
- ✅ **Metadata Tracking**: Tracks who performed the rollback and when
- ✅ **Concurrency Control**: Prevents race conditions during rollback

## Usage

### Automatic Rollback (to Previous Release)

```yaml
name: Rollback App

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options:
          - staging
          - production

jobs:
  rollback:
    uses: carlssonk/cicd-toolkit/.github/workflows/rollback-s3.yml@v1
    with:
      environment: ${{ inputs.environment }}
      aws_region: us-east-1
      aws_role_arn: arn:aws:iam::123456789012:role/github-actions-role
      s3_bucket: my-app-bucket
    permissions:
      id-token: write
      contents: read
```

### Manual Rollback (to Specific Commit)

```yaml
name: Rollback App

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options:
          - staging
          - production
      commit_hash:
        type: string
        description: "Commit hash to rollback to"

jobs:
  rollback:
    uses: carlssonk/cicd-toolkit/.github/workflows/rollback-s3.yml@v1
    with:
      environment: ${{ inputs.environment }}
      aws_region: us-east-1
      aws_role_arn: arn:aws:iam::123456789012:role/github-actions-role
      s3_bucket: my-app-bucket
      target_commit_hash: ${{ inputs.commit_hash }}
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

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `target_commit_hash` | Specific commit to rollback to | `""` (auto-detect previous) |
| `cache_control_mutable` | Cache-Control for latest path | `public, max-age=300` |
| `s3_additional_args` | Additional aws s3 sync args | `""` |
| `enable_cloudflare_cache_purge` | Enable Cloudflare cache purge | `false` |
| `cloudflare_zone_id` | Cloudflare Zone ID | `""` |

## Outputs

| Output | Description |
|--------|-------------|
| `target_commit_hash` | The commit hash that was rolled back to |
| `previous_commit_hash` | The commit hash that was rolled back from |
| `rollback_mode` | Rollback mode (`auto` or `manual`) |

## How It Works

### Automatic Rollback

1. Reads current deployment metadata from S3
2. Extracts `prev-commit-hash-short` from metadata
3. Verifies the previous version exists in S3
4. Syncs the previous version to the latest path
5. Updates metadata with rollback information

### Manual Rollback

1. Validates the provided commit hash against git history
2. Verifies the commit exists in S3 (was previously deployed)
3. Syncs that version to the latest path
4. Updates metadata with rollback information

### Metadata Updates

After rollback, the latest path metadata includes:
- `commit-hash-short`: Target commit (rolled back to)
- `prev-commit-hash-short`: Previous commit (rolled back from)
- `rolled-back-from`: Commit we rolled back from
- `rollback-by`: GitHub username who performed rollback
- `deployed-at`: Rollback timestamp

## Prerequisites

### AWS Setup

Same as [Deploy to S3](./deploy-s3.md#aws-setup)

### Deployment Requirements

The rollback workflow requires:
1. Previous deployment was done using `deploy-s3.yml`
2. Versioned deployments exist in S3 (`s3://bucket/{commit-hash}/`)
3. Latest path has proper metadata

## Examples

### List Available Versions

To see what versions are available for rollback:

```bash
aws s3 ls s3://my-app-bucket/ | grep "PRE" | awk '{print $2}' | sed 's/\///'
```

### Complete Rollback Workflow

```yaml
name: Rollback App

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        description: "Environment to rollback"
        required: true
        options:
          - staging
          - production
      commit_hash:
        type: string
        description: "Optional: Specific commit hash to rollback to"
        required: false

jobs:
  rollback:
    uses: carlssonk/cicd-toolkit/.github/workflows/rollback-s3.yml@v1
    with:
      environment: ${{ inputs.environment }}
      aws_region: ${{ vars.AWS_REGION }}
      aws_role_arn: arn:aws:iam::${{ vars.AWS_ACCOUNT_ID }}:role/github-actions-role
      s3_bucket: ${{ vars.S3_BUCKET }}
      target_commit_hash: ${{ inputs.commit_hash }}
    permissions:
      id-token: write
      contents: read

  notify:
    needs: rollback
    if: always()
    uses: carlssonk/cicd-toolkit/.github/workflows/notify.yml@v1
    with:
      notification_type: slack
      status: ${{ needs.rollback.result }}
      title: "Rollback ${{ inputs.environment }}"
      environment: ${{ inputs.environment }}
      commit_hash: ${{ needs.rollback.outputs.target_commit_hash }}
      slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Best Practices

1. **Test in Staging First**: Always test rollback in staging before production
2. **Verify Before Rolling Back**: Check the target version is correct
3. **Communicate**: Notify your team before rolling back production
4. **Monitor After Rollback**: Watch metrics/logs after rollback
5. **Document**: Document why the rollback was necessary

## Troubleshooting

### "No previous commit hash found"

This means:
- This is the first deployment, OR
- The deployment was made before metadata tracking was added

**Solution**: Use manual rollback with a specific commit hash

### "Commit not found in S3"

The specified commit was never deployed or has been deleted.

**Solution**: List available versions and choose one that exists

### "Invalid commit hash"

The commit doesn't exist in your git history.

**Solution**: Verify the commit hash is correct and exists in your repository

## Safety Features

- ✅ Validates commit exists in git history
- ✅ Verifies version exists in S3 before rolling back
- ✅ Requires full git history for validation
- ✅ Tracks who performed the rollback
- ✅ Concurrency control prevents simultaneous rollbacks

## Cloudflare Cache Purge

To automatically purge Cloudflare cache after rollback:

```yaml
jobs:
  rollback:
    uses: carlssonk/cicd-toolkit/.github/workflows/rollback-s3.yml@v1
    with:
      environment: production
      aws_region: us-east-1
      aws_role_arn: arn:aws:iam::123456789012:role/github-actions-role
      s3_bucket: my-app-bucket
      enable_cloudflare_cache_purge: true
      cloudflare_zone_id: ${{ vars.CLOUDFLARE_ZONE_ID }}
    secrets:
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    permissions:
      id-token: write
      contents: read
```

## Related Workflows

- [Deploy to S3](./deploy-s3.md)
- [Create Release](./create-release.md)
- [Send Notification](./notify.md)

