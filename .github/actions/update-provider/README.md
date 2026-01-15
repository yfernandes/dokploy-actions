# Update Dokploy Docker Provider

Update the Docker provider configuration for Dokploy applications. Set Docker image, registry credentials, and image references.

## Features

✅ Update Docker image reference  
✅ Configure private Docker registries  
✅ Set registry credentials  
✅ Direct API integration  
✅ Detailed error messages  
✅ Works with any Docker registry  

## Usage

### Basic - Update Docker Image

```yaml
- uses: ./.github/actions/update-provider
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: 'ghcr.io/user/app:latest'
```

### With Registry Credentials

```yaml
- uses: ./.github/actions/update-provider
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: 'registry.example.com/app:latest'
    registry_url: 'registry.example.com'
    username: ${{ secrets.REGISTRY_USERNAME }}
    password: ${{ secrets.REGISTRY_PASSWORD }}
```

### Marketplace Usage

```yaml
- uses: yago/dokploy-actions/update-provider@v1
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: 'ghcr.io/myorg/app:${{ github.sha }}'
```

## Inputs

| Input | Description | Required | Example |
|-------|-------------|----------|---------|
| `dokploy_url` | Dokploy instance URL | ✅ Yes | `https://dokploy.example.com` |
| `api_key` | Dokploy API key | ✅ Yes | - |
| `app_id` | Dokploy application ID | ✅ Yes | - |
| `docker_image` | Docker image reference | ✅ Yes | `ghcr.io/user/app:latest` |
| `registry_url` | Private registry URL | ❌ No | `registry.example.com` |
| `username` | Registry username | ❌ No | - |
| `password` | Registry password | ❌ No | - |

## Outputs

| Output | Description |
|--------|-------------|
| `success` | Whether the update succeeded (`true`/`false`) |
| `error_message` | Error description if update failed |

## Docker Image Formats

### Docker Hub
```yaml
docker_image: 'myimage:latest'
docker_image: 'myimage:v1.0.0'
docker_image: 'username/myimage:latest'
```

### GitHub Container Registry (GHCR)
```yaml
docker_image: 'ghcr.io/myorg/app:latest'
docker_image: 'ghcr.io/username/repo:main'
```

### Private Registry
```yaml
docker_image: 'registry.example.com/app:latest'
registry_url: 'registry.example.com'
username: ${{ secrets.REGISTRY_USER }}
password: ${{ secrets.REGISTRY_PASS }}
```

### AWS ECR
```yaml
docker_image: '123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest'
registry_url: '123456789.dkr.ecr.us-east-1.amazonaws.com'
username: 'AWS'
password: ${{ secrets.ECR_PASSWORD }}
```

## Examples

### Update Image from Build

```yaml
name: Update Docker Image

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker Image
        run: |
          docker build -t ghcr.io/myorg/app:${{ github.sha }} .
          docker push ghcr.io/myorg/app:${{ github.sha }}
        env:
          REGISTRY_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Update Provider
        uses: ./.github/actions/update-provider
        with:
          dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
          api_key: ${{ secrets.DOKPLOY_API_KEY }}
          app_id: ${{ secrets.DOKPLOY_APP_ID }}
          docker_image: 'ghcr.io/myorg/app:${{ github.sha }}'
```

### Update and Deploy

```yaml
- name: Update Provider
  id: update
  uses: ./.github/actions/update-provider
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: 'ghcr.io/myorg/app:latest'

- name: Deploy
  if: steps.update.outputs.success == 'true'
  uses: ./.github/actions/dokploy-deploy
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: 'ghcr.io/myorg/app:latest'
```

### Private Registry with Credentials

```yaml
- name: Update Private Registry
  uses: ./.github/actions/update-provider
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: 'myregistry.example.com/app:${{ github.ref_name }}'
    registry_url: 'myregistry.example.com'
    username: ${{ secrets.REGISTRY_USERNAME }}
    password: ${{ secrets.REGISTRY_PASSWORD }}
```

## Error Handling

The action fails with meaningful error messages for:
- Invalid API key (HTTP 401/403)
- Application not found (HTTP 404)
- Malformed Docker image reference
- Invalid registry credentials
- Network/connection errors

## Requirements

- curl (available on GitHub Actions runners)
- jq (available on GitHub Actions runners)
- Valid Dokploy API key
- Application exists in Dokploy

## Troubleshooting

**"Invalid API key" or HTTP 401**
→ Verify your Dokploy API key is correct
→ Ensure API key has necessary permissions

**"Application not found" (HTTP 404)**
→ Double-check `app_id` matches a real application
→ Verify the application exists in Dokploy dashboard

**Registry credentials not working**
→ Test credentials locally: `docker login registry.example.com`
→ Ensure credentials are stored in GitHub Secrets
→ Check registry URL matches image registry

**Image pull fails after update**
→ Verify Docker image exists in registry
→ Check that image is accessible from Dokploy server
→ Verify registry credentials are correct
→ Check image name spelling and tag

## Tips

- Store registry credentials in GitHub Secrets, never hardcode
- Test with non-production applications first
- Use specific image tags (not just `latest`)
- Combine with `dokploy-deploy` action for full workflow
- Monitor Dokploy logs for image pull issues

## See Also

- [Dokploy Deploy](../dokploy-deploy) - Deploy after updating provider
- [Preview Deployment](../preview-deployment) - Create preview environments
- [Dokploy API Documentation](https://dokploy.io/docs/api)
