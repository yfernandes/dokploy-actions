# Dokploy Deploy - Marketplace Metadata

This file contains metadata for publishing this action to the GitHub Marketplace.

## Action Details

- **Name:** Dokploy Deploy
- **Description:** Deploy to Dokploy and poll for completion
- **Category:** Deployment
- **Branding:**
  - Color: `blue`
  - Icon: `arrow-right` (or customize as needed)

## Marketplace Keywords

- dokploy
- deployment
- deploy
- ci-cd
- docker
- automation

## Versioning Strategy

This action uses semantic versioning:
- **Major.Minor.Patch** (e.g., `v1.0.0`)
- Create release tags as: `dokploy-deploy-v1.0.0`

## Publishing Steps

1. Ensure `action.yml` is up-to-date
2. Create a git tag: `git tag dokploy-deploy-v1.0.0`
3. Push the tag: `git push origin dokploy-deploy-v1.0.0`
4. Create a GitHub Release with the tag
5. The action will appear in Marketplace automatically

## Marketplace URL (after publishing)

`https://github.com/marketplace/actions/dokploy-deploy`

## Support

For issues or questions, open an issue in the repository.
