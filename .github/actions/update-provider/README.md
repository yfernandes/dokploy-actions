# Update Dokploy Provider

Update application provider configuration in Dokploy. Supports Docker, Git, Compose, and Raw providers.

## Features

✅ Update Docker image reference  
✅ Update Git repository configuration  
✅ Update Docker Compose configuration  
✅ Update Raw provider settings  
✅ Generic provider type support  
✅ JSON payload validation  
✅ Detailed error messages  

## Usage

### Update Docker Image

```yaml
- uses: ./.github/actions/update-provider
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    provider_type: 'docker'
    provider_data: '{"dockerImage": "ghcr.io/user/app:latest"}'
```

### Update Git Repository

```yaml
- uses: ./.github/actions/update-provider
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    provider_type: 'git'
    provider_data: '{"repository": "https://github.com/user/repo.git", "branch": "main"}'
```

### Update Docker Compose

```yaml
- uses: ./.github/actions/update-provider
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    provider_type: 'compose'
    provider_data: '{"composeFile": "docker-compose.yml", "composeFileContent": "..."}'
```

### From Marketplace

```yaml
- uses: yago/dokploy-actions/update-provider@v1
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    provider_type: 'docker'
    provider_data: '{"dockerImage": "ghcr.io/user/app:${{ github.sha }}"}'
```

## Inputs

| Input | Description | Required | Example |
|-------|-------------|----------|---------|
| `dokploy_url` | Dokploy instance URL | ✅ Yes | `https://dokploy.example.com` |
| `api_key` | Dokploy API key | ✅ Yes | - |
| `app_id` | Dokploy application ID | ✅ Yes | - |
| `provider_type` | Provider type | ✅ Yes | `docker`, `git`, `compose`, `raw` |
| `provider_data` | Provider configuration JSON | ✅ Yes | `{"dockerImage": "..."}` |

## Outputs

| Output | Description |
|--------|-------------|
| `success` | Whether the update succeeded (`true`/`false`) |
| `response` | Full API response from Dokploy |
| `error_message` | Error description if update failed |

## Supported Provider Types

### Docker
Update Docker image reference:
```json
{"dockerImage": "ghcr.io/user/app:latest"}
```

### Git
Update Git repository:
```json
{
  "repository": "https://github.com/user/repo.git",
  "branch": "main",
  "buildCommand": "npm run build"
}
```

### Compose
Update Docker Compose configuration:
```json
{
  "composeFile": "docker-compose.yml",
  "composeFileContent": "version: '3'..."
}
```

### Raw
Update Raw provider (for custom configurations):
```json
{"rawConfiguration": "..."}
```

## Error Handling

The action fails with meaningful error messages for:
- Invalid JSON in `provider_data`
- Invalid `provider_type`
- API authentication errors (HTTP 401/403)
- Application not found (HTTP 404)
- API server errors (HTTP 5xx)

## Examples

### Multi-Step Workflow

```yaml
name: Update and Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Update Docker Provider
        id: update
        uses: ./.github/actions/update-provider
        with:
          dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
          api_key: ${{ secrets.DOKPLOY_API_KEY }}
          app_id: ${{ secrets.DOKPLOY_APP_ID }}
          provider_type: 'docker'
          provider_data: '{"dockerImage": "ghcr.io/myapp:${{ github.sha }}"}'
      
      - name: Check Result
        run: |
          if [ "${{ steps.update.outputs.success }}" = "true" ]; then
            echo "✅ Provider updated successfully"
          else
            echo "❌ Update failed: ${{ steps.update.outputs.error_message }}"
            exit 1
          fi
```

### Using Environment Variables for JSON

```yaml
- name: Update Provider with Dynamic Config
  uses: ./.github/actions/update-provider
  env:
    PROVIDER_DATA: |
      {
        "dockerImage": "ghcr.io/myapp:latest",
        "environmentVariables": [
          {"key": "NODE_ENV", "value": "production"}
        ]
      }
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    provider_type: 'docker'
    provider_data: '${{ env.PROVIDER_DATA }}'
```

## Requirements

- curl (available on most GitHub Actions runners)
- jq (available on most GitHub Actions runners)
- Valid Dokploy API key
- Application exists in Dokploy

## Troubleshooting

**"Invalid JSON in provider_data"**
→ Ensure your JSON is valid. Use `jq` locally to validate: `echo '...' | jq`

**"Unknown provider type"**
→ Check that `provider_type` is one of: `docker`, `git`, `compose`, `raw`

**"HTTP 401" or "API key error"**
→ Verify your Dokploy API key is correct and has necessary permissions

**"Application not found" (HTTP 404)**
→ Double-check `app_id` matches a real application in Dokploy

## Tips

- Validate JSON in provider_data before passing to this action
- Use this action in combination with `dokploy-deploy` to update and deploy in one workflow
- Store sensitive configuration in GitHub Secrets
- Test with a non-production application first

## See Also

- [Dokploy Deploy](../dokploy-deploy) - Deploy after updating provider
- [Preview Deployment](../preview-deployment) - Create preview environments
- [Dokploy API Documentation](https://dokploy.io/docs/api)
