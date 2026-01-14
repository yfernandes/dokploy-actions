# Dokploy Actions

A collection of GitHub Actions for automating deployments and operations with [Dokploy](https://dokploy.io/).

## 📦 Available Actions

### 1. [Dokploy Deploy](./actions/dokploy-deploy)

Deploy applications to Dokploy and poll for completion.

**Features:**
- ✅ Updates Docker image reference in Dokploy
- ✅ Triggers deployment via Dokploy API
- ✅ Polls deployment status until completion
- ✅ Configurable timeout
- ✅ Error handling with meaningful messages

**Usage:**
```yaml
- uses: yago/dokploy-actions/dokploy-deploy@v1
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: ghcr.io/user/app:latest
```

[→ Full Documentation](./.github/actions/dokploy-deploy/README.md)

---

### 2. [Update Provider](./actions/update-provider)

Update application provider configuration in Dokploy (Docker, Git, Compose, etc.).

**Features:**
- ✅ Generic provider type support (docker, git, compose, raw)
- ✅ JSON-based configuration
- ✅ Validation of provider data
- ✅ Support for multiple provider types from one action
- ✅ Detailed error messages

**Usage:**
```yaml
- uses: yago/dokploy-actions/update-provider@v1
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    provider_type: 'docker'
    provider_data: '{"dockerImage": "ghcr.io/user/app:latest"}'
```

[→ Full Documentation](./.github/actions/update-provider/README.md)

---

### 3. [Preview Deployment](./actions/preview-deployment)

Create or update preview deployments in Dokploy for pull requests.

**Features:**
- ✅ Automatically create preview applications per PR
- ✅ Update Docker image for preview
- ✅ Poll deployment status
- ✅ Optional cleanup on failure
- ✅ Links to preview environment
- ✅ Perfect for testing before merge

**Usage:**
```yaml
- uses: yago/dokploy-actions/preview-deployment@v1
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: ghcr.io/user/app:pr-${{ github.event.pull_request.number }}
    pr_number: ${{ github.event.pull_request.number }}
    pr_title: ${{ github.event.pull_request.title }}
```

[→ Full Documentation](./.github/actions/preview-deployment/README.md)

---

### 4. [Migrate Database](./actions/migrate-database)

Run database migrations in Dokploy applications via command execution.

**Features:**
- ✅ Support for any migration framework
- ✅ Configurable timeout
- ✅ Automatic rollback on failure (optional)
- ✅ Execution time tracking
- ✅ Detailed command output
- ✅ Works with npm, python, go, php, and more

**Usage:**
```yaml
- uses: yago/dokploy-actions/migrate-database@v1
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    migration_command: 'npm run migrate'
    rollback_on_failure: 'true'
    rollback_command: 'npm run migrate:rollback'
```

[→ Full Documentation](./.github/actions/migrate-database/README.md)

---

## 🚀 Using These Actions

### From GitHub Marketplace
Each action is published to the GitHub Marketplace and can be referenced directly:
```yaml
- uses: yago/dokploy-actions/dokploy-deploy@v1
```

### From Source (during development)
```yaml
- uses: ./.github/actions/dokploy-deploy
```

## 🛠️ Development

### Repository Structure
```
.
├── .github/
│   ├── actions/
│   │   ├── dokploy-deploy/
│   │   │   ├── action.yml
│   │   │   ├── README.md
│   │   │   └── ...
│   │   └── [more actions...]
│   └── workflows/
├── README.md (you are here)
└── [documentation files]
```

### Creating a New Action

1. Create a new directory under `.github/actions/`:
   ```bash
   mkdir -p .github/actions/my-action
   ```

2. Add `action.yml` with your action definition

3. Create `README.md` with usage documentation

4. Test locally with [act](https://github.com/nektos/act) or by triggering workflows

### Testing Actions Locally
See [TESTING_WITH_ACT.md](.github/TESTING_WITH_ACT.md)

## 📋 Publishing to GitHub Marketplace

Each action can be published to the Marketplace independently:

1. **Tag the release** using action-specific tags:
   ```bash
   git tag dokploy-deploy-v1.0.0
   git push origin dokploy-deploy-v1.0.0
   ```

2. **Create a release** on GitHub with the tag

3. **Update your action's metadata** in the [GitHub App settings](https://github.com/settings/apps)

4. **Verify it appears** in the GitHub Marketplace

> **Note:** You can have multiple actions in one repository. Each gets its own marketplace listing!

## 📝 Documentation

- [Quick Reference Guide](.github/QUICK_REFERENCE.md)
- [Docker Build Workflow](.github/DOCKER_BUILD_WORKFLOW.md)
- [Implementation Summary](.github/IMPLEMENTATION_SUMMARY.md)
- [Local Testing Guide](.github/TESTING_WITH_ACT.md)

## 🔐 Security

- Never commit secrets to the repository
- Use GitHub Secrets for sensitive values
- Validate all inputs in action implementations

## 📄 License

[Add your license here]

## 🤝 Contributing

Contributions are welcome! Please ensure:
- [ ] New actions follow the structure of existing actions
- [ ] `action.yml` is properly formatted
- [ ] `README.md` includes usage examples
- [ ] Changes are tested locally with `act`

