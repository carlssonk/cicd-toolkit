# Create Release Tag

A reusable GitHub Actions workflow for creating release tags and GitHub releases with automatic changelog generation.

## Features

- ✅ **Automatic Tagging**: Creates git tags based on commit hashes
- ✅ **GitHub Releases**: Optionally creates GitHub releases
- ✅ **Changelog Generation**: Generates changelogs from conventional commits
- ✅ **Flexible Tagging**: Customizable tag prefixes and formats
- ✅ **Idempotent**: Safely handles existing tags
- ✅ **Environment-Aware**: Can restrict releases to specific environments

## Usage

### Basic Example

```yaml
name: Deploy and Release

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: your-org/github-workflows-library/.github/workflows/deploy-to-s3.yml@v1
    with:
      environment: production
      # ... other inputs
    permissions:
      id-token: write
      contents: read

  create-release:
    needs: deploy
    uses: your-org/github-workflows-library/.github/workflows/create-release.yml@v1
    with:
      environment: production
      commit_hash: ${{ needs.deploy.outputs.commit_hash }}
    permissions:
      contents: write
```

### With Custom Tag Prefix

```yaml
jobs:
  create-release:
    uses: your-org/github-workflows-library/.github/workflows/create-release.yml@v1
    with:
      environment: production
      commit_hash: ${{ github.sha }}
      tag_prefix: v
      create_release: true
      release_as_latest: true
    permissions:
      contents: write
```

### With Changelog Generation

```yaml
jobs:
  create-release:
    uses: your-org/github-workflows-library/.github/workflows/create-release.yml@v1
    with:
      environment: production
      commit_hash: ${{ github.sha }}
      generate_changelog: true
      changelog_types: "feat,fix,perf,refactor,docs"
    permissions:
      contents: write
```

### For Staging (Pre-release)

```yaml
jobs:
  create-release:
    uses: your-org/github-workflows-library/.github/workflows/create-release.yml@v1
    with:
      environment: staging
      commit_hash: ${{ github.sha }}
      only_production: false
      release_as_prerelease: true
      release_as_latest: false
    permissions:
      contents: write
```

## Inputs

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `environment` | Environment being deployed | `production` |
| `commit_hash` | Commit hash being deployed | `abc12345` |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `tag_prefix` | Prefix for tag name | `app` |
| `create_release` | Create GitHub release | `true` |
| `release_as_latest` | Mark as latest release | `true` |
| `release_as_prerelease` | Mark as prerelease | `false` |
| `generate_changelog` | Generate changelog | `true` |
| `changelog_types` | Commit types for changelog | `feat,fix,perf,refactor` |
| `only_production` | Only create for production | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `tag_name` | The tag name that was created |
| `tag_existed` | Whether the tag already existed |
| `release_url` | URL of the created GitHub release |

## Tag Format

Tags are created in the format: `{prefix}-{commit_hash}`

Examples:
- `tag_prefix: "app"` → `app-abc12345`
- `tag_prefix: "v"` → `v-abc12345`
- `tag_prefix: "release"` → `release-abc12345`

## Changelog Generation

The workflow generates changelogs from conventional commits:

### Supported Commit Types

| Type | Icon | Description |
|------|------|-------------|
| `feat` | ✨ | New features |
| `fix` | 🐛 | Bug fixes |
| `perf` | ⚡ | Performance improvements |
| `refactor` | ♻️ | Code refactoring |
| `docs` | 📚 | Documentation changes |
| `test` | 🧪 | Test changes |
| `chore` | 🔧 | Chore/maintenance |

### Commit Format

The workflow recognizes these formats:
- `feat: add new feature`
- `feat(scope): add new feature`
- `fix: fix bug`
- `fix(api): fix endpoint`

### Example Changelog

```markdown
## 📝 Changes

### ✨ Features
- abc1234 feat: add user authentication
- def5678 feat(api): add new endpoint

### 🐛 Bug Fixes
- ghi9012 fix: resolve login issue
- jkl3456 fix(ui): correct button alignment

### ⚡ Performance
- mno7890 perf: optimize database queries

### 🔀 Other Changes
- pqr1234 update dependencies
```

## Release Notes

Release notes include:

1. **Deployment Information**
   - Environment (Production/Staging)
   - Commit hash
   - Commit message
   - Author
   - Date

2. **Changelog** (if enabled)
   - Grouped by commit type
   - With commit hashes
   - Links to commits

## Examples

### Complete Deployment Pipeline

```yaml
name: Deploy and Release

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: your-org/github-workflows-library/.github/workflows/deploy-to-s3.yml@v1
    with:
      environment: production
      aws_region: us-east-1
      aws_role_arn: ${{ secrets.AWS_ROLE_ARN }}
      s3_bucket: my-app-bucket
      build_command: npm run build
      build_output_dir: dist
    permissions:
      id-token: write
      contents: read

  create-release:
    needs: deploy
    if: needs.deploy.result == 'success'
    uses: your-org/github-workflows-library/.github/workflows/create-release.yml@v1
    with:
      environment: production
      commit_hash: ${{ needs.deploy.outputs.commit_hash }}
      generate_changelog: true
    permissions:
      contents: write

  notify:
    needs: [deploy, create-release]
    if: always()
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: slack
      status: ${{ needs.deploy.result }}
      title: "Deployment Complete"
      environment: production
      commit_hash: ${{ needs.deploy.outputs.commit_hash }}
      deployment_url: ${{ needs.create-release.outputs.release_url }}
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Multi-Environment with Different Strategies

```yaml
jobs:
  deploy-staging:
    uses: your-org/github-workflows-library/.github/workflows/deploy-to-s3.yml@v1
    with:
      environment: staging
      # ... other inputs

  release-staging:
    needs: deploy-staging
    uses: your-org/github-workflows-library/.github/workflows/create-release.yml@v1
    with:
      environment: staging
      commit_hash: ${{ needs.deploy-staging.outputs.commit_hash }}
      only_production: false
      release_as_prerelease: true
      release_as_latest: false
      tag_prefix: staging
    permissions:
      contents: write

  deploy-production:
    needs: deploy-staging
    uses: your-org/github-workflows-library/.github/workflows/deploy-to-s3.yml@v1
    with:
      environment: production
      # ... other inputs

  release-production:
    needs: deploy-production
    uses: your-org/github-workflows-library/.github/workflows/create-release.yml@v1
    with:
      environment: production
      commit_hash: ${{ needs.deploy-production.outputs.commit_hash }}
      tag_prefix: v
    permissions:
      contents: write
```

## Best Practices

1. **Use Conventional Commits**: Follow conventional commit format for better changelogs
2. **Pin Workflow Versions**: Use specific versions when referencing this workflow
3. **Production Only**: Consider restricting releases to production deployments
4. **Semantic Versioning**: For libraries, consider using semantic version tags
5. **Release Notes**: Review and edit release notes after creation if needed

## Troubleshooting

### Tag Already Exists

The workflow is idempotent - if a tag already exists, it will skip creation without error.

### No Changelog Generated

Ensure:
1. You have previous tags to compare against
2. Your commits follow conventional commit format
3. `generate_changelog` is set to `true`

### Permission Denied

Ensure the workflow has `contents: write` permission.

## Related Workflows

- [Deploy to S3](./deploy-to-s3.md)
- [Rollback S3 Deployment](./rollback-s3.md)
- [Send Notification](./notify.md)

