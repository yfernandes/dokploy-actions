# Publishing Actions to GitHub Marketplace

This guide explains how to publish multiple GitHub Actions from this repository to the GitHub Marketplace.

## Overview

You can publish multiple independent actions from a single repository. Each action:
- Gets its own Marketplace listing
- Uses semantic versioning with action-specific tags
- Has independent release history

## Step-by-Step Publishing

### 1. Prepare Your Action

Before publishing, ensure your action is ready:

```yaml
# .github/actions/my-action/action.yml
name: 'My Action Name'
description: 'Clear description of what it does'
branding:
  icon: 'arrow-right'  # or any icon from https://feathericons.com/
  color: 'blue'        # Available colors: white, yellow, blue, green, red, orange, purple

inputs:
  # ... input definitions
outputs:
  # ... output definitions
runs:
  # ... implementation
```

### 2. Create Action-Specific Tags

Use tags that identify which action they belong to:

```bash
# Format: <action-name>-v<major>.<minor>.<patch>
git tag dokploy-deploy-v1.0.0
git push origin dokploy-deploy-v1.0.0
```

### 3. Create a GitHub Release

Go to your repository's Releases page:

1. Click "Draft a new release"
2. Select your tag (e.g., `dokploy-deploy-v1.0.0`)
3. Set title: `Dokploy Deploy v1.0.0`
4. Add release notes describing changes
5. Publish the release

### 4. Marketplace Listing

The action will **automatically appear** in the Marketplace after 24 hours with:
- Name from `action.yml`
- Description from `action.yml`
- Branding icon and color
- Your repository information
- Release notes

## Versioning Strategy for Multiple Actions

To manage versions cleanly when you have multiple actions:

### Option A: Action-Specific Tags (Recommended)
```
dokploy-deploy-v1.0.0    ← dokploy-deploy action
dokploy-deploy-v1.1.0
my-other-action-v1.0.0   ← my-other-action action
```

**Advantage:** Clear which action each tag refers to

### Option B: Monorepo Versioning
```
v1.0.0  ← all actions bump together
v1.1.0
```

**Advantage:** Single version for entire repo
**Disadvantage:** Less flexibility if actions evolve at different rates

## Usage After Publishing

Once published, users can reference your actions:

```yaml
# From GitHub Marketplace
- uses: yago/dokploy-actions/dokploy-deploy@v1.0.0
- uses: yago/dokploy-actions/my-other-action@v1.0.0

# Or use minor version to auto-update patches
- uses: yago/dokploy-actions/dokploy-deploy@v1
```

## Managing Updates

When you make updates to an action:

1. Update files in `.github/actions/<action-name>/`
2. Commit changes to `main` or your release branch
3. Create a new tag: `dokploy-deploy-v1.0.1`
4. Push the tag
5. Create a release with the tag
6. Action updates are available within ~15 minutes

## Marketplace Visibility

Your actions will appear in the Marketplace with:
- **Direct URL:** `https://github.com/marketplace/actions/dokploy-deploy`
- **Searchable:** By action name, keywords in `action.yml`
- **Rating:** Community can rate and review

## Best Practices

✅ **DO:**
- Keep action names descriptive and unique
- Update `action.yml` before releasing
- Include detailed `README.md` with examples
- Use semantic versioning strictly
- Test actions before releasing
- Write clear release notes

❌ **DON'T:**
- Use generic names like "Deploy" or "Build"
- Publish without documentation
- Mix changes from multiple actions in one release
- Break backward compatibility in patch versions
- Skip testing

## Example Workflow: Publishing a New Action

```bash
# 1. Create the action
mkdir -p .github/actions/new-action
cat > .github/actions/new-action/action.yml << 'YAML'
name: 'New Action'
description: 'Does something useful'
runs:
  using: 'composite'
  steps:
    - run: echo "Hello!"
      shell: bash
YAML

# 2. Create README with examples
cat > .github/actions/new-action/README.md << 'MD'
# New Action

Usage:
\`\`\`yaml
- uses: yago/dokploy-actions/new-action@v1
\`\`\`
MD

# 3. Commit
git add .github/actions/new-action/
git commit -m "Add: new-action"

# 4. Tag and push
git tag new-action-v1.0.0
git push origin new-action-v1.0.0

# 5. Create release on GitHub
# → Releases → Draft new release → Select tag → Publish
```

## Troubleshooting

**Action doesn't appear in Marketplace after 24 hours:**
- Verify `action.yml` is in the correct directory
- Check that action has a valid name and description
- Ensure repository is public
- Try searching by exact action name

**"This action isn't compatible with this version":**
- Confirm users are using `@v1` or `@v1.0.0` format
- Check that tag matches the format

**Need to unpublish an action:**
- Delete the tag: `git tag -d dokploy-deploy-v1.0.0 && git push origin :dokploy-deploy-v1.0.0`
- Remove the release on GitHub
- It will be removed from Marketplace within 24 hours

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Marketplace](https://github.com/marketplace?category=&type=actions)
- [Creating Actions](https://docs.github.com/en/actions/creating-actions)
- [Metadata Syntax](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions)
