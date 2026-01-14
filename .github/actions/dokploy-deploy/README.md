# Dokploy Deploy Action

A GitHub Action for deploying applications to Dokploy and polling for completion.

## Features

✅ Updates Docker image reference in Dokploy  
✅ Triggers deployment via Dokploy API  
✅ Polls deployment status until completion  
✅ Configurable timeout  
✅ Error handling with meaningful messages  
✅ Works with any Dokploy instance  

## Usage

### Basic Example

```yaml
- uses: ./.github/actions/dokploy-deploy
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_ADMIN_APP_ID }}
    docker_image: ghcr.io/greenbudz/medusa-admin:latest
```

### With Custom Deployment Info

```yaml
- uses: ./.github/actions/dokploy-deploy
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_ADMIN_APP_ID }}
    docker_image: ghcr.io/greenbudz/medusa-admin:latest
    deployment_title: "Deploy from main (${{ github.sha }})"
    deployment_description: "Automated deployment triggered by push"
    max_wait_minutes: 30
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `dokploy_url` | Dokploy instance URL | ✅ Yes | - |
| `api_key` | Dokploy API key | ✅ Yes | - |
| `app_id` | Dokploy application ID | ✅ Yes | - |
| `docker_image` | Docker image to deploy | ✅ Yes | - |
| `deployment_title` | Deployment title for logs | ❌ No | `Automated Deployment` |
| `deployment_description` | Deployment description | ❌ No | `` |
| `max_wait_minutes` | Max wait time in minutes | ❌ No | `25` |

## Outputs

| Output | Description |
|--------|-------------|
| `deployment_status` | Final deployment status: `success`, `failed`, or `timeout` |

## Example: Using Output

```yaml
- uses: ./.github/actions/dokploy-deploy
  id: deploy
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_ADMIN_APP_ID }}
    docker_image: ghcr.io/greenbudz/medusa-admin:latest

- name: Check Deployment Status
  run: |
    if [ "${{ steps.deploy.outputs.deployment_status }}" = "success" ]; then
      echo "✅ Deployment succeeded!"
    else
      echo "❌ Deployment failed or timed out"
    fi
```

## How It Works

1. **Update Docker Provider**
   - Calls `POST /api/application.saveDockerProvider`
   - Sets the Docker image reference in Dokploy

2. **Trigger Deployment**
   - Calls `POST /api/application.redeploy`
   - Starts the deployment process

3. **Wait for Completion**
   - Polls `GET /api/deployment.all?applicationId=APP_ID`
   - Checks status every 10 seconds
   - Waits until deployment completes or timeout reached

## Deployment Status Polling

The action polls deployment status with these intervals:
- Check every **10 seconds**
- Maximum wait: **25 minutes** (configurable via `max_wait_minutes`)
- Success states: `success`, `done`
- Failure states: `failed`, `error`
- Pending states: `in_progress`, `running`, `pending`

## Error Handling

The action fails with meaningful error messages for:
- Invalid API key (HTTP 401/403)
- Application not found (HTTP 404)
- Deployment failure in Dokploy
- Timeout after max_wait_minutes

## Example Workflow

```yaml
name: Deploy to Dokploy

on:
  push:
    branches: [main, releases]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy Admin to Dokploy
        uses: ./.github/actions/dokploy-deploy
        with:
          dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
          api_key: ${{ secrets.DOKPLOY_API_KEY }}
          app_id: ${{ secrets.DOKPLOY_ADMIN_APP_ID }}
          docker_image: ghcr.io/greenbudz/medusa-admin:latest
          deployment_title: "Deployment from ${{ github.ref_name }} (${{ github.sha }})"
          deployment_description: "Triggered by: ${{ github.event.head_commit.message }}"
```

## Requirements

- curl (available on most GitHub Actions runners)
- jq (available on most GitHub Actions runners)
- Valid Dokploy API key
- Dokploy application configured with Docker build type

## Troubleshooting

**"API key error" in logs**
→ Verify `api_key` input is correct

**"Application not found" error**
→ Double-check `app_id` matches Dokploy dashboard

**Deployment never completes**
→ Increase `max_wait_minutes` if deployments take longer
→ Check Dokploy logs for actual build/deployment errors

**HTTP 200 but deployment fails**
→ Check Dokploy logs for Docker image pull errors
→ Verify Docker image exists and is accessible
→ Check Docker registry credentials in Dokploy settings

## Development

To modify the action, edit `action.yml` in this directory.

### Testing
```bash
# Manually trigger a workflow that uses this action
# See: .github/workflows/dokploy-deploy.yml
```

## License

Same as parent repository
