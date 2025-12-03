# Send Notification

A reusable GitHub Actions workflow for sending notifications to various platforms (Slack, Discord, Microsoft Teams, GitHub Discussions).

## Features

- ✅ **Multiple Platforms**: Slack, Discord, Microsoft Teams, GitHub Discussions
- ✅ **Rich Formatting**: Status-based colors and emojis
- ✅ **Flexible Content**: Include commit info, workflow links, deployment URLs
- ✅ **Status-Aware**: Different formatting for success, failure, warning
- ✅ **Mention on Failure**: Alert team members when things go wrong

## Usage

### Slack Notification

```yaml
jobs:
  deploy:
    # ... deployment job

  notify:
    needs: deploy
    if: always()
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: slack
      status: ${{ needs.deploy.result }}
      title: "Production Deployment"
      message: "Application deployed successfully"
      environment: production
      commit_hash: ${{ needs.deploy.outputs.commit_hash }}
      slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Discord Notification

```yaml
jobs:
  notify:
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: discord
      status: success
      title: "Build Complete"
      message: "All tests passed"
      discord_webhook_url: ${{ secrets.DISCORD_WEBHOOK_URL }}
```

### Microsoft Teams Notification

```yaml
jobs:
  notify:
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: teams
      status: failure
      title: "Deployment Failed"
      message: "Check the logs for details"
      environment: production
      teams_webhook_url: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

### GitHub Discussion

```yaml
jobs:
  notify:
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: github-discussion
      status: success
      title: "New Production Release"
      message: "Version 2.0 is now live!"
      environment: production
      commit_hash: abc12345
      github_discussion_category: "Announcements"
    permissions:
      discussions: write
```

### With Mention on Failure

```yaml
jobs:
  deploy:
    # ... deployment job

  notify:
    needs: deploy
    if: always()
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: slack
      status: ${{ needs.deploy.result }}
      title: "Production Deployment"
      environment: production
      mention_on_failure: "@channel"  # or "@here" for Discord
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Inputs

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `notification_type` | Platform to notify | `slack`, `discord`, `teams`, `github-discussion` |
| `status` | Workflow status | `success`, `failure`, `cancelled`, `warning` |
| `title` | Notification title | `"Production Deployment"` |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `message` | Notification message | `""` |
| `environment` | Environment name | `""` |
| `commit_hash` | Commit hash | `""` |
| `deployment_url` | Deployment URL | `""` |
| `slack_webhook_url` | Slack webhook URL | `""` |
| `discord_webhook_url` | Discord webhook URL | `""` |
| `teams_webhook_url` | Teams webhook URL | `""` |
| `github_discussion_category` | Discussion category | `"Announcements"` |
| `include_workflow_link` | Include workflow run link | `true` |
| `include_commit_info` | Include commit details | `true` |
| `mention_on_failure` | Mention string for failures | `""` |

### Secrets

| Secret | Description |
|--------|-------------|
| `SLACK_WEBHOOK_URL` | Slack webhook URL (alternative to input) |
| `DISCORD_WEBHOOK_URL` | Discord webhook URL (alternative to input) |
| `TEAMS_WEBHOOK_URL` | Teams webhook URL (alternative to input) |

## Status Types

| Status | Emoji | Color | Use Case |
|--------|-------|-------|----------|
| `success` | ✅ | Green | Successful deployments/builds |
| `failure` | ❌ | Red | Failed deployments/builds |
| `cancelled` | ⚠️ | Yellow | Cancelled workflows |
| `warning` | ⚠️ | Yellow | Warnings or partial success |

## Platform-Specific Setup

### Slack

1. Create a Slack App: https://api.slack.com/apps
2. Enable Incoming Webhooks
3. Create a webhook for your channel
4. Add webhook URL to GitHub secrets

**Webhook Format**: `https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX`

### Discord

1. Open Discord channel settings
2. Go to Integrations → Webhooks
3. Create a new webhook
4. Copy webhook URL
5. Add to GitHub secrets

**Webhook Format**: `https://discord.com/api/webhooks/123456789/abcdefghijklmnop`

### Microsoft Teams

1. Open Teams channel
2. Click "..." → Connectors
3. Configure "Incoming Webhook"
4. Copy webhook URL
5. Add to GitHub secrets

**Webhook Format**: `https://outlook.office.com/webhook/...`

### GitHub Discussions

1. Enable Discussions in repository settings
2. Create categories (e.g., "Announcements")
3. Ensure workflow has `discussions: write` permission

## Examples

### Complete Deployment Pipeline with Notifications

```yaml
name: Deploy and Notify

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
    if: needs.deploy.result == 'success'
    uses: your-org/github-workflows-library/.github/workflows/create-release.yml@v1
    with:
      environment: production
      commit_hash: ${{ needs.deploy.outputs.commit_hash }}
    permissions:
      contents: write

  notify-slack:
    needs: [deploy, create-release]
    if: always()
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: slack
      status: ${{ needs.deploy.result }}
      title: "Production Deployment"
      message: ${{ needs.deploy.result == 'success' && 'Deployment completed successfully!' || 'Deployment failed. Please check the logs.' }}
      environment: production
      commit_hash: ${{ needs.deploy.outputs.commit_hash }}
      deployment_url: ${{ needs.create-release.outputs.release_url }}
      mention_on_failure: "@channel"
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

  notify-discussion:
    needs: [deploy, create-release]
    if: needs.deploy.result == 'success'
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: github-discussion
      status: success
      title: "🚀 New Production Release"
      message: "A new version has been deployed to production!"
      environment: production
      commit_hash: ${{ needs.deploy.outputs.commit_hash }}
      deployment_url: ${{ needs.create-release.outputs.release_url }}
    permissions:
      discussions: write
```

### Multi-Platform Notifications

```yaml
jobs:
  deploy:
    # ... deployment job

  notify-slack:
    needs: deploy
    if: always()
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: slack
      status: ${{ needs.deploy.result }}
      title: "Deployment ${{ needs.deploy.result }}"
      environment: production
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

  notify-discord:
    needs: deploy
    if: always()
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: discord
      status: ${{ needs.deploy.result }}
      title: "Deployment ${{ needs.deploy.result }}"
      environment: production
    secrets:
      DISCORD_WEBHOOK_URL: ${{ secrets.DISCORD_WEBHOOK_URL }}

  notify-teams:
    needs: deploy
    if: always()
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: teams
      status: ${{ needs.deploy.result }}
      title: "Deployment ${{ needs.deploy.result }}"
      environment: production
    secrets:
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

### Conditional Notifications

```yaml
jobs:
  deploy:
    # ... deployment job

  notify-on-failure:
    needs: deploy
    if: needs.deploy.result == 'failure'
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: slack
      status: failure
      title: "🚨 Production Deployment Failed"
      message: "Immediate attention required!"
      environment: production
      mention_on_failure: "@channel"
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

  notify-on-success:
    needs: deploy
    if: needs.deploy.result == 'success'
    uses: your-org/github-workflows-library/.github/workflows/notify.yml@v1
    with:
      notification_type: github-discussion
      status: success
      title: "✅ Production Deployment Successful"
      environment: production
    permissions:
      discussions: write
```

## Best Practices

1. **Use Secrets**: Always store webhook URLs in GitHub secrets
2. **Conditional Notifications**: Use `if: always()` to notify on both success and failure
3. **Appropriate Mentions**: Use `@channel` sparingly, only for critical failures
4. **Rich Context**: Include environment, commit hash, and deployment URLs
5. **Multiple Channels**: Consider different notifications for different audiences

## Troubleshooting

### Slack: "Invalid Webhook"

- Verify webhook URL is correct
- Check webhook hasn't been revoked
- Ensure URL is in GitHub secrets

### Discord: Message Not Appearing

- Verify webhook URL format
- Check channel permissions
- Ensure webhook hasn't been deleted

### Teams: "Bad Request"

- Verify webhook URL is correct
- Check JSON payload is valid
- Ensure connector is still active

### GitHub Discussions: Permission Denied

- Ensure workflow has `discussions: write` permission
- Verify Discussions are enabled in repository
- Check category name exists

## Related Workflows

- [Deploy to S3](./deploy-to-s3.md)
- [Rollback S3 Deployment](./rollback-s3.md)
- [Create Release](./create-release.md)

