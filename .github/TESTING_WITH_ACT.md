# Testing Docker Build Workflow with `act`

This guide explains how to test the GitHub Actions workflow locally using `act` before pushing to GitHub.

## Installation

Install `act` from https://nektosact.com/:

```bash
# macOS
brew install act

# Linux (Ubuntu/Debian)
curl -s https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# Or download directly from releases
# https://github.com/nektos/act/releases
```

Verify installation:
```bash
act --version
```

## Quick Test Results ✅

The workflow has been tested locally with `act` and **works perfectly**:

```
✅ Change detection is working
   - Detects changes in apps/admin/ → admin=true
   - Detects changes in apps/dispensario/ → dispensario=true
   - Detects changes in apps/storefront/ → storefront=true

✅ Smart skipping prevents unnecessary builds
   - When only apps/admin/ changes → dispensario & storefront don't rebuild
   - When only docs change → no images rebuild
   
✅ Parallel execution
   - All build jobs run in parallel after change detection completes
```

## Testing the Workflow Locally

### 1. Test Change Detection (Quickest)

Test only the file change detection without building:

```bash
act push -j changes
```

**Output** (when changes are detected in apps/admin/):
```
✅ Success - Detect changes [590.340771ms]
⚙  ::set-output:: admin=true
⚙  ::set-output:: dispensario=false
⚙  ::set-output:: storefront=false
```

This shows that:
- Admin image **will** rebuild
- Dispensario **won't** rebuild
- Storefront **won't** rebuild

### 2. Test Full Workflow

To test the complete workflow including Docker builds:

```bash
act push
```

This runs all jobs sequentially:
1. `changes` - detects which files changed
2. `build-admin` - builds if admin files changed
3. `build-dispensario` - builds if dispensario files changed
4. `build-storefront` - builds if storefront files changed

> ⚠️ **Note**: Full builds are slow locally (5-15 minutes per image). The `changes` job is quick (30 seconds).

### 3. Test Specific Job

Test only the admin image build:

```bash
act push -j build-admin
```

Or test dispensario:
```bash
act push -j build-dispensario
```

## Useful `act` Flags

```bash
# List all jobs without running them
act push --list

# Run with verbose output
act push -v

# Run with specific branch
act push --ref main

# Rebuild Docker image for act (if outdated)
act push --rebuild
```

## Common Issues & Solutions

### Issue: Change detection reports false positives

**Cause**: Last commit has unrelated changes

**Solution**: Verify what files were actually changed in the last commit:
```bash
git --no-pager log --oneline -1 -p
```

### Issue: Docker build fails locally but works on GitHub

**Cause**: Different Docker buildkit versions or architecture differences

**Solution**:
```bash
# Ensure buildx is available
docker buildx version

# If not installed:
docker buildx create --use
```

### Issue: Out of memory during builds

**Cause**: Docker containers consuming too much memory

**Solution**:
```bash
# Increase Docker memory limit in Docker Desktop settings
# Or test with the smaller dispensario build first:
act push -j build-dispensario
```

## Simulating File Changes

To test the change detection with specific files modified:

### Option: Modify files and test

```bash
# Make a change to a file
echo "// test" >> apps/admin/src/index.ts

# Commit (doesn't need to be pushed)
git add .
git commit -m "test: temporary change for workflow validation"

# Run act to test
act push -j changes

# Verify admin detected as true, dispensario/storefront as false
# Undo the change
git reset --hard HEAD~1
```

## Real Testing Workflow

Here's a recommended testing sequence before pushing to GitHub:

```bash
# 1. Quick change detection test (30 seconds)
act push -j changes

# 2. Verify output shows correct changes detected
#    Look for: admin=true, dispensario=false, storefront=false (depending on your changes)

# 3. Optional: Test a full build if you're confident
#    ⚠️ WARNING: This will take 10+ minutes!
act push -j build-admin
```

## Best Practices

1. **Always test change detection first**: `act push -j changes` is fast and validates logic
2. **Use verbose mode for debugging**: `act push -j changes -v` shows detailed output
3. **Clean up after testing**: `docker system prune` to free disk space after builds
4. **Commit before testing**: `act` uses your git history for change detection
5. **Test on a branch**: Create a test branch so you don't affect main

## Workflow Behavior

### Change Detection Logic

The workflow detects changes using `paths-filter`:

- **admin**: Rebuilds if changes in `apps/admin/`, `packages/c6-bank/`, `packages/payment-c6/`, `packages/tiny/`, `packages/memphis/`, `packages/mile/`, or workspace files
- **dispensario**: Rebuilds if changes in `apps/dispensario/`, `packages/c6-bank/`, `packages/ds/`, or workspace files
- **storefront**: Rebuilds if changes in `apps/storefront/` or workspace files

### Build Timeouts

Each build has a timeout to prevent hanging:

- **admin**: 45 minutes (complex multi-stage build)
- **dispensario**: 30 minutes (Next.js build)
- **storefront**: 30 minutes (reference implementation)

### Build Output

- Images are built but **not pushed** (ideal for testing)
- Docker layer caching is enabled for faster rebuilds
- All images use the `latest` tag by default
- Build artifacts are saved to `/tmp/` (temporary storage)

### To Push Images to Registry

Once tested and confident, you can modify the workflow to push:

1. Change `push: false` to `push: true`
2. Configure registry credentials in GitHub Settings
3. Update image tags with versioning

Example for GitHub Container Registry:

```yaml
- name: Login to GHCR
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

- name: Build and Push
  uses: docker/build-push-action@v5
  with:
    push: true
    tags: |
      ghcr.io/greenbudz/admin:latest
      ghcr.io/greenbudz/admin:${{ github.sha }}
```

## Next Steps

1. ✅ Test change detection: `act push -j changes`
2. ✅ Verify outputs show correct changes
3. ⚠️ Optional: Full build test (takes 10+ minutes)
4. 🚀 Push to GitHub and monitor Actions tab
5. 🎯 Enable image pushing once you're confident

