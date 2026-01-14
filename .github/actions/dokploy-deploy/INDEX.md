# Dokploy Deploy Action - Complete Guide

## 📦 What's Included

This GitHub Action automates deployments to Dokploy with intelligent status polling.

**Files:**
- `action.yml` - Action definition
- `README.md` - Usage guide
- Main workflow: `.github/workflows/dokploy-deploy.yml`
- Full docs: `ops/DOKPLOY_DEPLOYMENT.md`

---

## 🚀 Quick Start

```yaml
- uses: ./.github/actions/dokploy-deploy
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_ADMIN_APP_ID }}
    docker_image: ghcr.io/greenbudz/medusa-admin:latest
```

---

## 📖 Documentation Map

| Document | Purpose | Read Time |
|----------|---------|-----------|
| **README.md** | Action usage & API reference | 10 min |
| **ops/DOKPLOY_DEPLOYMENT.md** | Full guide + future roadmap | 20 min |

---

## 🔑 Key Features

✅ Updates Docker image in Dokploy  
✅ Triggers deployment via API  
✅ Polls status every 10 seconds  
✅ Configurable timeout  
✅ Error handling  

---

## 📋 What Changed

**Before**: 195 lines of workflow YAML with inline curl commands  
**After**: 45 lines of clean workflow + 150 lines of reusable action

**Benefits:**
- ✅ Cleaner workflow file
- ✅ Reusable across projects
- ✅ Easier to test
- ✅ Easier to maintain
- ✅ Can be published to GitHub Marketplace

---

## 🔄 How Workflows Reference This

```yaml
# .github/workflows/dokploy-deploy.yml
- uses: ./.github/actions/dokploy-deploy
  with:
    dokploy_url: ...
    api_key: ...
    app_id: ...
    docker_image: ...
```

---

## 📚 Next Steps

1. **First time?** → Read `README.md`
2. **Setting up?** → See `ops/DOKPLOY_DEPLOYMENT.md` → Quick Start
3. **Custom needs?** → Edit `action.yml` inputs/outputs
4. **Ready to share?** → Publish to GitHub Marketplace

---

## 🎯 Moving to Personal Repo

To move this action to your personal GitHub account:

1. Create repo: `github.com/YOUR_USERNAME/dokploy-deploy-action`
2. Copy files:
   - `action.yml`
   - `README.md`
   - `LICENSE` (optional)
3. Update workflow reference:
   ```yaml
   - uses: YOUR_USERNAME/dokploy-deploy-action@v1
   ```

---

**Status**: ✅ Ready to use or share  
**Last Updated**: 2026-01-14
