# Invalidate Cloudflare Cache

A reusable GitHub Actions workflow for purging Cloudflare cache.

## Features

- ✅ **Purge Everything**: Clear all cached content for a zone
- ✅ **Selective Purge**: Purge specific URLs, prefixes, hosts, or cache tags
- ✅ **Enterprise Support**: Supports prefix and tag-based purging (Enterprise only)
- ✅ **Secure**: Uses Cloudflare API tokens for authentication

## Usage

### Basic Example (Purge Everything)

```yaml
name: Purge Cache

on:
  workflow_dispatch:

jobs:
  purge:
    uses: carlssonk/cicd-toolkit/.github/workflows/invalidate-cloudflare-cache.yml@v1
    with:
      cloudflare_zone_id: ${{ vars.CLOUDFLARE_ZONE_ID }}
      purge_everything: true
    secrets:
      cloudflare_api_token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

### Purge Specific URLs

```yaml
jobs:
  purge:
    uses: carlssonk/cicd-toolkit/.github/workflows/invalidate-cloudflare-cache.yml@v1
    with:
      cloudflare_zone_id: ${{ vars.CLOUDFLARE_ZONE_ID }}
      purge_everything: false
      purge_urls: '["https://example.com/latest/index.html", "https://example.com/latest/app.js"]'
    secrets:
      cloudflare_api_token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

### Purge by Hostname

```yaml
jobs:
  purge:
    uses: carlssonk/cicd-toolkit/.github/workflows/invalidate-cloudflare-cache.yml@v1
    with:
      cloudflare_zone_id: ${{ vars.CLOUDFLARE_ZONE_ID }}
      purge_everything: false
      purge_hosts: '["example.com", "www.example.com"]'
    secrets:
      cloudflare_api_token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

## Inputs

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `cloudflare_zone_id` | Cloudflare Zone ID | `abc123...` |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `purge_everything` | Purge all cached content | `true` |
| `purge_urls` | JSON array of specific URLs to purge | `""` |
| `purge_prefixes` | JSON array of URL prefixes (Enterprise) | `""` |
| `purge_hosts` | JSON array of hostnames to purge | `""` |
| `purge_tags` | JSON array of cache tags (Enterprise) | `""` |

### Required Secrets

| Secret | Description |
|--------|-------------|
| `cloudflare_api_token` | Cloudflare API token with cache purge permissions |

## Outputs

| Output | Description |
|--------|-------------|
| `purge_id` | The ID of the purge request |
| `purge_type` | Type of purge performed |

## Purge Methods

### Purge Everything

Clears all cached content for the zone. This is the simplest but most aggressive option.

```yaml
purge_everything: true
```

### Purge by URLs

Purge specific URLs. Useful for targeted cache invalidation.

```yaml
purge_everything: false
purge_urls: '["https://example.com/page1", "https://example.com/page2"]'
```

### Purge by Prefixes (Enterprise Only)

Purge all URLs matching a prefix. Requires Cloudflare Enterprise.

```yaml
purge_everything: false
purge_prefixes: '["https://example.com/static/"]'
```

### Purge by Hosts

Purge all cached content for specific hostnames.

```yaml
purge_everything: false
purge_hosts: '["api.example.com"]'
```

### Purge by Cache Tags (Enterprise Only)

Purge content by cache tags. Requires Cloudflare Enterprise and Cache-Tag headers.

```yaml
purge_everything: false
purge_tags: '["tag1", "tag2"]'
```

## Prerequisites

### Cloudflare Setup

1. **Zone ID**: Find your Zone ID in the Cloudflare dashboard under Overview
2. **API Token**: Create an API token with the following permissions:
   - Zone → Cache Purge → Purge

### Creating an API Token

1. Go to Cloudflare Dashboard → My Profile → API Tokens
2. Click "Create Token"
3. Use the "Custom token" template
4. Set permissions:
   - Zone → Cache Purge → Purge
5. Set zone resources:
   - Include → Specific zone → Your zone
6. Create and copy the token

### GitHub Setup

1. Add the API token as a repository secret:
   - Name: `CLOUDFLARE_API_TOKEN`
   - Value: Your Cloudflare API token

2. Add the Zone ID as a repository variable:
   - Name: `CLOUDFLARE_ZONE_ID`
   - Value: Your Cloudflare Zone ID

## Integration with Deploy/Rollback

The `deploy-s3.yml` and `rollback-s3.yml` workflows have built-in Cloudflare cache purging:

```yaml
jobs:
  deploy:
    uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1
    with:
      # ... other inputs
      enable_cloudflare_cache_purge: true
      cloudflare_zone_id: ${{ vars.CLOUDFLARE_ZONE_ID }}
    secrets:
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

For more control, you can use this workflow separately after deployment:

```yaml
jobs:
  deploy:
    uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1
    with:
      # ... deployment inputs (without cache purge)
    permissions:
      id-token: write
      contents: read

  purge-cache:
    needs: deploy
    uses: carlssonk/cicd-toolkit/.github/workflows/invalidate-cloudflare-cache.yml@v1
    with:
      cloudflare_zone_id: ${{ vars.CLOUDFLARE_ZONE_ID }}
      purge_everything: false
      purge_urls: '["https://example.com/latest/index.html"]'
    secrets:
      cloudflare_api_token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

## Best Practices

1. **Use Selective Purging**: When possible, purge only what changed instead of everything
2. **Protect Your Token**: Never expose your API token in logs or code
3. **Rate Limits**: Be aware of Cloudflare API rate limits (1000 requests/5 minutes)
4. **Cache Tags**: For large sites, consider using cache tags for granular purging

## Troubleshooting

### "Authentication error"

- Verify your API token is correct
- Ensure the token has Cache Purge permissions
- Check the token hasn't expired

### "Zone not found"

- Verify your Zone ID is correct
- Ensure the API token has access to the zone

### "Rate limit exceeded"

- Cloudflare limits purge requests to 1000 per 5 minutes
- Consider using `purge_everything` instead of many individual URLs

## Related Workflows

- [Deploy to S3](./deploy-s3.md)
- [Rollback S3 Deployment](./rollback-s3.md)

