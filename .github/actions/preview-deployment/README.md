# Preview Deployment

Create or update preview deployments in Dokploy for pull requests. Perfect for testing code changes before merging to main.

## Features

✅ Automatically create preview applications  
✅ Update Docker image for preview  
✅ Trigger preview deployment  
✅ Poll deployment completion  
✅ Optional cleanup on failure  
✅ Links to preview environment  
✅ Works with any Dokploy instance  

## Usage

### Basic Example

```yaml
name: Preview Deployment

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build and Push Docker Image
        run: |
          docker build -t ghcr.io/myorg/app:pr-${{ github.event.pull_request.number }} .
          docker push ghcr.io/myorg/app:pr-${{ github.event.pull_request.number }}
      
      - name: Deploy Preview
        uses: ./.github/actions/preview-deployment
        with:
          dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
          api_key: ${{ secrets.DOKPLOY_API_KEY }}
          app_id: ${{ secrets.DOKPLOY_PRODUCTION_APP_ID }}
          docker_image: ghcr.io/myorg/app:pr-${{ github.event.pull_request.number }}
          pr_number: ${{ github.event.pull_request.number }}
          pr_title: ${{ github.event.pull_request.title }}
          github_repo: ${{ github.repository }}
```

### With Auto-Cleanup

```yaml
- uses: ./.github/actions/preview-deployment
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: ghcr.io/myorg/app:pr-${{ github.event.pull_request.number }}
    pr_number: ${{ github.event.pull_request.number }}
    pr_title: ${{ github.event.pull_request.title }}
    cleanup_on_failure: 'true'
```

### Marketplace Usage

```yaml
- uses: yago/dokploy-actions/preview-deployment@v1
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: ghcr.io/myorg/app:pr-${{ github.event.pull_request.number }}
    pr_number: ${{ github.event.pull_request.number }}
    pr_title: ${{ github.event.pull_request.title }}
```

## Inputs

| Input | Description | Required | Example |
|-------|-------------|----------|---------|
| `dokploy_url` | Dokploy instance URL | ✅ Yes | `https://dokploy.example.com` |
| `api_key` | Dokploy API key | ✅ Yes | - |
| `app_id` | Template app ID | ✅ Yes | - |
| `docker_image` | Docker image for preview | ✅ Yes | `ghcr.io/org/app:pr-123` |
| `pr_number` | Pull request number | ✅ Yes | `${{ github.event.pull_request.number }}` |
| `pr_title` | Pull request title | ✅ Yes | `${{ github.event.pull_request.title }}` |
| `github_repo` | GitHub repo (owner/repo) | ❌ No | `${{ github.repository }}` |
| `deployment_name` | Name prefix for preview | ❌ No | `preview` (default) |
| `max_wait_minutes` | Max wait for deployment | ❌ No | `25` (default) |
| `cleanup_on_failure` | Delete preview on failure | ❌ No | `false` (default) |

## Outputs

| Output | Description |
|--------|-------------|
| `deployment_status` | Status: `success`, `failed`, or `timeout` |
| `preview_url` | URL to access the preview deployment |
| `deployment_id` | Dokploy deployment ID |

## How It Works

1. **Fetch Template App** - Gets source application configuration
2. **Create/Find Preview App** - Creates `preview-pr-{number}` app if needed
3. **Update Docker Image** - Sets the Docker image for preview
4. **Trigger Deployment** - Starts the deployment process
5. **Poll Status** - Waits for deployment to complete
6. **Return Results** - Provides preview URL and status

## Naming Convention

Preview applications are named using the pattern:
```
{deployment_name}-pr-{pr_number}
```

Default example: `preview-pr-123`

## Comment on Pull Request

To add a comment with preview URL to the PR:

```yaml
- name: Deploy Preview
  id: preview
  uses: ./.github/actions/preview-deployment
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: ghcr.io/myorg/app:pr-${{ github.event.pull_request.number }}
    pr_number: ${{ github.event.pull_request.number }}
    pr_title: ${{ github.event.pull_request.title }}

- name: Comment Preview URL
  if: steps.preview.outputs.deployment_status == 'success'
  uses: actions/github-script@v7
  with:
    script: |
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: '🚀 Preview deployment is ready!\n\n[Visit Preview](${{ steps.preview.outputs.preview_url }})'
      })
```

## Cleanup Preview Deployments

To clean up old preview deployments, consider adding:

```yaml
name: Cleanup Closed PR Previews

on:
  pull_request:
    types: [closed]

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Delete Preview
        run: |
          curl -X DELETE "${{ secrets.DOKPLOY_BASE_URL }}/api/application.remove?applicationId=$PREVIEW_APP_ID" \
            -H "x-api-key: ${{ secrets.DOKPLOY_API_KEY }}"
        env:
          PREVIEW_APP_ID: "preview-pr-${{ github.event.pull_request.number }}"
```

## Error Handling

The action fails if:
- Template application not found
- Docker image update fails
- Deployment triggers but then fails
- Timeout before deployment completes

With `cleanup_on_failure: true`, the preview application is deleted if deployment fails.

## Examples

### Full CI/CD Pipeline

```yaml
name: PR Validation & Preview

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  build-and-preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install & Build
        run: |
          npm ci
          npm run build
      
      - name: Build Docker Image
        run: |
          docker build -t ghcr.io/myorg/app:pr-${{ github.event.pull_request.number }} .
          docker push ghcr.io/myorg/app:pr-${{ github.event.pull_request.number }}
        env:
          REGISTRY_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Deploy Preview
        id: preview
        uses: ./.github/actions/preview-deployment
        with:
          dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
          api_key: ${{ secrets.DOKPLOY_API_KEY }}
          app_id: ${{ secrets.DOKPLOY_APP_ID }}
          docker_image: ghcr.io/myorg/app:pr-${{ github.event.pull_request.number }}
          pr_number: ${{ github.event.pull_request.number }}
          pr_title: ${{ github.event.pull_request.title }}
          github_repo: ${{ github.repository }}
          cleanup_on_failure: 'true'
      
      - name: Comment Preview Link
        if: steps.preview.outputs.deployment_status == 'success'
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '✅ Preview deployment successful!\n\n🔗 [Preview Environment](${{ steps.preview.outputs.preview_url }})'
            })
      
      - name: Notify Failure
        if: steps.preview.outputs.deployment_status != 'success'
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '❌ Preview deployment failed'
            })
```

## Requirements

- curl (available on GitHub Actions runners)
- jq (available on GitHub Actions runners)
- Valid Dokploy API key
- Docker image pushed to accessible registry

## Troubleshooting

**"Failed to fetch application details"**
→ Verify `app_id` is correct and API key has access

**Preview app created but deployment fails**
→ Check Docker image exists and is accessible
→ Verify registry credentials in Dokploy

**Timeout**
→ Increase `max_wait_minutes`
→ Check Dokploy logs for build/deployment issues

## See Also

- [Dokploy Deploy](../dokploy-deploy) - Deploy to production
- [Update Provider](../update-provider) - Update application configuration
- [Migrate Database](../migrate-database) - Run database migrations
