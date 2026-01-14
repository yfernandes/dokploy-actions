# Docker Build Workflow Guide

## Overview

The `.github/workflows/docker-build.yml` workflow automatically builds Docker images when relevant files change in the repository. This prevents unnecessary rebuilds and speeds up CI/CD.

## How It Works

### File Change Detection

The workflow uses `dorny/paths-filter` to detect which files have changed and only builds affected images:

| Image | Rebuilds When Changes Detected In |
|-------|-----------------------------------|
| **admin** | `apps/admin/`, `packages/c6-bank/`, `packages/payment-c6/`, `packages/tiny/`, `packages/memphis/`, `packages/mile/`, workspace files |
| **dispensario** | `apps/dispensario/`, `packages/c6-bank/`, `packages/ds/`, workspace files |
| **storefront** | `apps/storefront/`, workspace files |

### Why These Dependencies?

- **admin** (Medusa backend) depends on all packages since it integrates C6 payments, Tiny ERP, Memphis/Mile shipping, and other integrations
- **dispensario** (Next.js storefront) depends on C6 Bank (payments) and DS (design system), but NOT on other backend packages
- **storefront** (reference implementation) is standalone with minimal dependencies

### Workflow Triggers

- On every **push** to `main` or `develop`
- On every **pull request** to `main` or `develop`

## Jobs

### 1. `changes` (Detect File Changes)

**Purpose**: Determine which Docker images need rebuilding

**Outputs**:
- `admin` - `true` if admin image needs rebuild
- `dispensario` - `true` if dispensario image needs rebuild
- `storefront` - `true` if storefront image needs rebuild

### 2. `build-admin`

**Condition**: Runs only if `admin` detected changes

**Actions**:
- Checks out repository
- Sets up Docker Buildx
- Builds `greenbudz/admin:latest` Docker image
- Uses GitHub Actions cache for faster rebuilds

### 3. `build-dispensario`

**Condition**: Runs only if `dispensario` detected changes

**Actions**:
- Checks out repository
- Sets up Docker Buildx
- Builds `greenbudz/dispensario:latest` Docker image
- Uses GitHub Actions cache for faster rebuilds

### 4. `build-storefront`

**Condition**: Runs only if `storefront` detected changes

**Actions**:
- Checks out repository
- Sets up Docker Buildx
- Builds `greenbudz/storefront:latest` Docker image
- Uses GitHub Actions cache for faster rebuilds

## Configuration

### Current Settings

```yaml
push: false           # Images are built but NOT pushed
tags: greenbudz/*:latest
cache: gha            # GitHub Actions cache (fast, no registry needed)
```

### To Enable Image Push

To push images to a registry (Docker Hub, ECR, GHCR):

1. Add `push: true` to each `build-push-action`:

```yaml
- name: Build and Push Admin Docker Image
  uses: docker/build-push-action@v5
  with:
    push: true  # Enable pushing
    tags: your-registry/admin:${{ github.sha }}
    # ... other settings
```

2. Configure registry credentials in GitHub Settings:
   - **Docker Hub**: Add `DOCKER_USERNAME` and `DOCKER_PASSWORD` secrets
   - **GitHub Container Registry**: Use default `GITHUB_TOKEN`
   - **AWS ECR**: Add `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`

3. Update `tags` to include registry and versioning:

```yaml
tags: |
  your-registry/admin:latest
  your-registry/admin:${{ github.sha }}
  your-registry/admin:${{ github.ref_name }}
```

## Local Testing with `act`

See [`TESTING_WITH_ACT.md`](./TESTING_WITH_ACT.md) for detailed instructions on testing locally with `act`.

Quick start:

```bash
# Install act
brew install act  # or see TESTING_WITH_ACT.md for Linux

# Test change detection
act push -j changes

# Test building a specific image
act push -j build-admin

# Test full workflow
act push
```

## Concurrency & Performance

The workflow includes concurrency settings:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

This means:
- Only one run per branch at a time
- New pushes cancel previous in-progress runs
- Prevents resource waste from overlapping builds

## Docker Layer Caching

The workflow uses GitHub Actions cache for Docker layers:

```yaml
cache-from: type=gha          # Read from cache
cache-to: type=gha,mode=max   # Save full cache (slower but most effective)
```

Benefits:
- ✅ Subsequent builds are much faster
- ✅ Cached across all workflow runs on the same branch
- ✅ No external registry needed

## Examples

### Scenario 1: Modified C6 Adapter Module

**Change**: `apps/admin/src/modules/c6-adapter/service.ts`

**Result**:
- ✅ `admin` image rebuilds (depends on `apps/admin/`)
- ❌ `dispensario` does NOT rebuild (doesn't depend on c6-adapter)
- ❌ `storefront` does NOT rebuild

### Scenario 2: Updated Design System Package

**Change**: `packages/ds/components/Button.tsx`

**Result**:
- ❌ `admin` does NOT rebuild (doesn't depend on `ds`)
- ✅ `dispensario` image rebuilds (depends on `packages/ds/`)
- ❌ `storefront` does NOT rebuild

### Scenario 3: Updated Workspace Lockfile

**Change**: `pnpm-lock.yaml`

**Result**:
- ✅ **All images rebuild** (depends on workspace files)

This is necessary since lockfile changes affect all packages.

### Scenario 4: Updated Documentation

**Change**: `README.md`, `docs/**`

**Result**:
- ❌ **No images rebuild** (documentation changes don't affect builds)

Perfect for reducing unnecessary CI runs!

## Troubleshooting

### Images aren't rebuilding when expected

**Check**:
1. Did the change match the paths filter? See the table above.
2. Is the workflow enabled? Check `.github/workflows/docker-build.yml` exists.
3. Are you on `main` or `develop` branch? (Workflow only runs on these branches)

### Build timeout

**Solutions**:
- Check Docker Buildx is properly configured
- Verify layer cache is being used (`cache-from: type=gha`)
- Consider splitting monolithic Dockerfiles

### Registry authentication fails

**Check**:
1. Are registry credentials configured in GitHub Settings?
2. Is the token/key still valid?
3. Does the account have push permissions to the repository?

## Next Steps

1. **Test locally**: Follow [`TESTING_WITH_ACT.md`](./TESTING_WITH_ACT.md)
2. **Enable push**: Add registry credentials and set `push: true`
3. **Add tagging**: Use git tags for release versioning
4. **Monitor**: Watch GitHub Actions tab for build status

## Related Documentation

- [GitHub Actions: Docker Build and Push](https://github.com/docker/build-push-action)
- [Medusa Docker Deployment](https://docs.medusajs.com/deployment/docker)
- [Next.js Docker Deployment](https://nextjs.org/docs/deployment/docker)
