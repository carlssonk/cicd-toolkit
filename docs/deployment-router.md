# Deployment Router

A reusable GitHub Actions workflow for routing deployments based on branching strategy and event type.

## Features

- ✅ **Multiple Strategies**: Supports trunk-based, GitFlow, GitHub Flow, and custom routing
- ✅ **Event-Aware**: Routes based on push, pull_request, or workflow_dispatch
- ✅ **Flexible Configuration**: Customizable branch patterns and environments
- ✅ **Action Determination**: Automatically determines whether to plan or deploy
- ✅ **Custom Routing**: JSON-based custom routing for complex scenarios

## Usage

### Trunk-Based Development (Default)

```yaml
name: Deploy App

on:
  push:
    branches: [main, dev/*]
  pull_request:
    branches: [main, dev/*]
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [dev, staging, production]

jobs:
  route:
    uses: carlssonk/cicd-toolkit/.github/workflows/deployment-router.yml@v1
    with:
      event_name: ${{ github.event_name }}
      branch: ${{ github.event_name == 'pull_request' && github.head_ref || github.ref_name }}
      workflow_dispatch_environment: ${{ github.event.inputs.environment }}

  deploy:
    needs: route
    if: needs.route.outputs.should_deploy == 'true'
    strategy:
      matrix:
        environment: ${{ fromJson(needs.route.outputs.environments) }}
    uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1
    with:
      environment: ${{ matrix.environment }}
      # ... other inputs
```

### GitFlow Strategy

```yaml
jobs:
  route:
    uses: carlssonk/cicd-toolkit/.github/workflows/deployment-router.yml@v1
    with:
      event_name: ${{ github.event_name }}
      branch: ${{ github.event_name == 'pull_request' && github.head_ref || github.ref_name }}
      deployment_strategy: gitflow
      dev_branch_pattern: "develop/*"
      staging_branch: "staging"
      production_branch: "main"
```

### GitHub Flow Strategy

```yaml
jobs:
  route:
    uses: carlssonk/cicd-toolkit/.github/workflows/deployment-router.yml@v1
    with:
      event_name: ${{ github.event_name }}
      branch: ${{ github.event_name == 'pull_request' && github.head_ref || github.ref_name }}
      deployment_strategy: github-flow
      production_branch: "main"
```

### Custom Routing

```yaml
jobs:
  route:
    uses: carlssonk/cicd-toolkit/.github/workflows/deployment-router.yml@v1
    with:
      event_name: ${{ github.event_name }}
      branch: ${{ github.event_name == 'pull_request' && github.head_ref || github.ref_name }}
      deployment_strategy: custom
      custom_routing: |
        {
          "main": {
            "push": {
              "environments": ["staging", "production"],
              "action": "deploy"
            },
            "pull_request": {
              "environments": ["staging", "production"],
              "action": "plan"
            }
          },
          "develop": {
            "push": {
              "environments": ["dev"],
              "action": "deploy"
            }
          },
          "feature/*": {
            "push": {
              "environments": ["preview"],
              "action": "deploy"
            }
          }
        }
```

## Inputs

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `event_name` | GitHub event name | `push`, `pull_request`, `workflow_dispatch` |
| `branch` | Branch name | `main`, `dev/feature-x` |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `workflow_dispatch_environment` | Environment from manual trigger | `""` |
| `workflow_dispatch_action` | Action from manual trigger | `""` |
| `deployment_strategy` | Deployment strategy | `trunk` |
| `dev_branch_pattern` | Pattern for dev branches | `dev/*` |
| `staging_branch` | Staging branch name | `staging` |
| `production_branch` | Production branch name | `main` |
| `custom_routing` | Custom routing JSON | `{}` |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `environments` | JSON array of environments | `["staging","production"]` |
| `action` | Action to perform | `deploy`, `plan`, `apply` |
| `should_deploy` | Whether to proceed | `true`, `false` |

## Deployment Strategies

### Trunk-Based Development

**Philosophy**: Main branch is always deployable, feature branches are short-lived.

**Routing Rules**:

| Event | Branch | Environments | Action | Deploy? |
|-------|--------|--------------|--------|---------|
| PR | `main` | staging, production | plan | No |
| PR | `dev/*` | dev | plan | No |
| Push | `main` | staging, production | deploy | Yes |
| Push | `dev/*` | dev | deploy | Yes |

**Best For**: Fast-moving teams, continuous deployment

### GitFlow

**Philosophy**: Separate branches for development, staging, and production.

**Routing Rules**:

| Event | Branch | Environments | Action | Deploy? |
|-------|--------|--------------|--------|---------|
| PR | `main` | production | plan | No |
| PR | `staging` | staging | plan | No |
| PR | `develop/*` | dev | plan | No |
| Push | `main` | production | deploy | Yes |
| Push | `staging` | staging | deploy | Yes |
| Push | `develop/*` | dev | deploy | Yes |

**Best For**: Regulated environments, scheduled releases

### GitHub Flow

**Philosophy**: Simple flow with main branch and feature branches.

**Routing Rules**:

| Event | Branch | Environments | Action | Deploy? |
|-------|--------|--------------|--------|---------|
| PR | `main` | production | plan | No |
| Push | `main` | production | deploy | Yes |

**Best For**: Simple projects, small teams

### Custom

**Philosophy**: Define your own routing logic.

**Format**:
```json
{
  "branch_name": {
    "event_type": {
      "environments": ["env1", "env2"],
      "action": "deploy"
    }
  }
}
```

## Examples

### Complete Trunk-Based Pipeline

```yaml
name: Deploy App

on:
  push:
    branches: [main, dev/*]
  pull_request:
    branches: [main, dev/*]
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [dev, staging, production]

jobs:
  route:
    uses: carlssonk/cicd-toolkit/.github/workflows/deployment-router.yml@v1
    with:
      event_name: ${{ github.event_name }}
      branch: ${{ github.event_name == 'pull_request' && github.head_ref || github.ref_name }}
      workflow_dispatch_environment: ${{ github.event.inputs.environment }}

  deploy-dev:
    needs: route
    if: |
      needs.route.outputs.should_deploy == 'true' &&
      contains(fromJson(needs.route.outputs.environments), 'dev')
    uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1
    with:
      environment: dev
      # ... other inputs

  deploy-staging:
    needs: route
    if: |
      needs.route.outputs.should_deploy == 'true' &&
      contains(fromJson(needs.route.outputs.environments), 'staging')
    uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1
    with:
      environment: staging
      # ... other inputs

  deploy-production:
    needs: [route, deploy-staging]
    if: |
      needs.route.outputs.should_deploy == 'true' &&
      contains(fromJson(needs.route.outputs.environments), 'production')
    uses: carlssonk/cicd-toolkit/.github/workflows/deploy-s3.yml@v1
    with:
      environment: production
      # ... other inputs
```

### GitFlow with Terraform

```yaml
name: Deploy Infrastructure

on:
  push:
    branches: [main, staging, develop/*]
  pull_request:
    branches: [main, staging, develop/*]

jobs:
  route:
    uses: carlssonk/cicd-toolkit/.github/workflows/deployment-router.yml@v1
    with:
      event_name: ${{ github.event_name }}
      branch: ${{ github.event_name == 'pull_request' && github.head_ref || github.ref_name }}
      deployment_strategy: gitflow
      workflow_dispatch_action: ${{ github.event.inputs.tf_action }}

  terraform-dev:
    needs: route
    if: contains(fromJson(needs.route.outputs.environments), 'dev')
    uses: carlssonk/terraform-workflows/.github/workflows/deploy.yml@v1
    with:
      environment: dev
      action: ${{ needs.route.outputs.action }}
      # ... other inputs
```

### Custom Routing for Microservices

```yaml
jobs:
  route:
    uses: carlssonk/cicd-toolkit/.github/workflows/deployment-router.yml@v1
    with:
      event_name: ${{ github.event_name }}
      branch: ${{ github.ref_name }}
      deployment_strategy: custom
      custom_routing: |
        {
          "main": {
            "push": {
              "environments": ["prod-us-east-1", "prod-eu-west-1", "prod-ap-southeast-1"],
              "action": "deploy"
            }
          },
          "staging": {
            "push": {
              "environments": ["staging-us-east-1"],
              "action": "deploy"
            }
          },
          "canary": {
            "push": {
              "environments": ["canary-prod"],
              "action": "deploy"
            }
          }
        }
```

## Best Practices

1. **Choose the Right Strategy**: Match your branching strategy to your team's workflow
2. **Use Matrix Deployments**: Deploy to multiple environments efficiently
3. **Add Approval Gates**: Use GitHub environments for production approvals
4. **Test in Lower Environments**: Always deploy to dev/staging before production
5. **Document Your Strategy**: Make sure your team understands the routing logic

## Troubleshooting

### No Environments Returned

Check:
1. Branch name matches your patterns
2. Event type is supported
3. Strategy is configured correctly

### Wrong Action Returned

Verify:
1. Event type (push vs pull_request)
2. Strategy matches your workflow
3. `workflow_dispatch_action` is set correctly for manual triggers

### Custom Routing Not Working

Ensure:
1. JSON is valid
2. Branch names match exactly (or use patterns)
3. Event names are correct

## Related Workflows

- [Deploy to S3](./deploy-s3.md)
- [Create Release](./create-release.md)
- [Send Notification](./notify.md)

