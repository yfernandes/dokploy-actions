# Migrate Database

Run database migrations in Dokploy applications. Supports any migration framework with command-line interface (npm, python, go, etc.).

## Features

✅ Execute migration commands in Dokploy apps  
✅ Support for any migration framework  
✅ Configurable timeout  
✅ Automatic rollback on failure (optional)  
✅ Execution time tracking  
✅ Detailed command output  
✅ Skip workflow on failure option  

## Usage

### NPM/Node.js Migrations

```yaml
- uses: ./.github/actions/migrate-database
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    migration_command: 'npm run migrate'
```

### Python/Django Migrations

```yaml
- uses: ./.github/actions/migrate-database
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    migration_command: 'python manage.py migrate'
```

### With Rollback Support

```yaml
- uses: ./.github/actions/migrate-database
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    migration_command: 'npm run migrate'
    rollback_on_failure: 'true'
    rollback_command: 'npm run migrate:rollback'
    timeout_seconds: '600'
```

### Marketplace Usage

```yaml
- uses: yago/dokploy-actions/migrate-database@v1
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    migration_command: 'npm run migrate'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `dokploy_url` | Dokploy instance URL | ✅ Yes | - |
| `api_key` | Dokploy API key | ✅ Yes | - |
| `app_id` | Dokploy application ID | ✅ Yes | - |
| `migration_command` | Migration command to run | ✅ Yes | - |
| `timeout_seconds` | Execution timeout | ❌ No | `300` |
| `skip_on_failure` | Continue if migration fails | ❌ No | `false` |
| `rollback_on_failure` | Rollback on failure | ❌ No | `false` |
| `rollback_command` | Rollback command | ❌ No | `` |

## Outputs

| Output | Description |
|--------|-------------|
| `migration_status` | Status: `success`, `failed`, or `timeout` |
| `migration_output` | Full output from the migration command |
| `execution_time` | Time taken in seconds |

## Supported Frameworks

### Node.js/NPM

**Sequelize:**
```yaml
migration_command: 'npm run migrate'
rollback_command: 'npm run migrate:undo'
```

**TypeORM:**
```yaml
migration_command: 'npm run typeorm migration:run'
rollback_command: 'npm run typeorm migration:revert'
```

**Knex:**
```yaml
migration_command: 'npm run knex migrate:latest'
rollback_command: 'npm run knex migrate:rollback'
```

### Python

**Django:**
```yaml
migration_command: 'python manage.py migrate'
rollback_command: 'python manage.py migrate app MIGRATION_NAME --backwards'
```

**Alembic:**
```yaml
migration_command: 'alembic upgrade head'
rollback_command: 'alembic downgrade -1'
```

**SQLAlchemy-migrate:**
```yaml
migration_command: 'migrate.py upgrade'
rollback_command: 'migrate.py downgrade'
```

### Go

**golang-migrate:**
```yaml
migration_command: 'migrate -path ./migrations -database "postgresql://..." up'
rollback_command: 'migrate -path ./migrations -database "postgresql://..." down'
```

**GORM:**
```yaml
migration_command: './app migrate'
rollback_command: './app migrate:rollback'
```

### PHP/Laravel

**Laravel Artisan:**
```yaml
migration_command: 'php artisan migrate'
rollback_command: 'php artisan migrate:rollback'
```

## Examples

### Deploy with Pre-Deployment Migration

```yaml
name: Deploy with Migrations

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Migrate Database
        uses: ./.github/actions/migrate-database
        with:
          dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
          api_key: ${{ secrets.DOKPLOY_API_KEY }}
          app_id: ${{ secrets.DOKPLOY_APP_ID }}
          migration_command: 'npm run migrate'
          timeout_seconds: '600'
      
      - name: Deploy Application
        uses: ./.github/actions/dokploy-deploy
        with:
          dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
          api_key: ${{ secrets.DOKPLOY_API_KEY }}
          app_id: ${{ secrets.DOKPLOY_APP_ID }}
          docker_image: ghcr.io/myorg/app:latest
```

### With Rollback on Failure

```yaml
- name: Migrate Database
  id: migrate
  uses: ./.github/actions/migrate-database
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    migration_command: 'npm run migrate'
    rollback_on_failure: 'true'
    rollback_command: 'npm run migrate:rollback'
    timeout_seconds: '300'

- name: Notify Migration Status
  if: always()
  run: |
    if [ "${{ steps.migrate.outputs.migration_status }}" = "success" ]; then
      echo "✅ Migration successful"
    else
      echo "❌ Migration failed after ${{ steps.migrate.outputs.execution_time }}s"
    fi
```

### Conditional Deployment

```yaml
- name: Migrate Database
  id: migrate
  uses: ./.github/actions/migrate-database
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    migration_command: 'npm run migrate'

- name: Deploy Only if Migration Successful
  if: steps.migrate.outputs.migration_status == 'success'
  uses: ./.github/actions/dokploy-deploy
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    docker_image: ghcr.io/myorg/app:latest
```

### Multiple Migrations

```yaml
- name: Migrate Main Database
  uses: ./.github/actions/migrate-database
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_APP_ID }}
    migration_command: 'npm run migrate:main'

- name: Migrate Analytics Database
  uses: ./.github/actions/migrate-database
  with:
    dokploy_url: ${{ secrets.DOKPLOY_BASE_URL }}
    api_key: ${{ secrets.DOKPLOY_API_KEY }}
    app_id: ${{ secrets.DOKPLOY_ANALYTICS_APP_ID }}
    migration_command: 'npm run migrate:analytics'
```

## Migration Best Practices

✅ **DO:**
- Always test migrations in a staging environment first
- Include rollback commands for production deployments
- Set appropriate timeout values for complex migrations
- Log migration output for debugging
- Use version control for migration files
- Test rollback procedures regularly
- Keep migrations atomic and idempotent

❌ **DON'T:**
- Skip migrations to save time
- Run migrations without backup
- Make migrations dependent on code that hasn't been deployed
- Forget to test rollback procedures
- Use generic commands without testing

## Troubleshooting

**"Migration timeout after Xs"**
→ Increase `timeout_seconds` for long-running migrations
→ Check for database locks or slow queries
→ Verify database connectivity

**"Migration failed with HTTP 500"**
→ Check Dokploy logs for execution errors
→ Verify migration command syntax
→ Ensure migration files exist in the container

**Rollback didn't execute**
→ Verify `rollback_command` is correct for your framework
→ Test rollback command locally first
→ Check application logs in Dokploy

**"Command not found" error**
→ Ensure migration command is available in the container
→ Check that dependencies are installed
→ Verify working directory is correct

## Requirements

- curl (available on GitHub Actions runners)
- jq (available on GitHub Actions runners)
- Valid Dokploy API key with execution permissions
- Migration framework configured in the application
- Database connectivity from the container

## Notes

- Migrations run inside the Dokploy application container
- The application must be running for migrations to execute
- Output is captured and returned in `migration_output`
- Execution time includes network overhead
- Rollback is optional and must be explicitly enabled

## See Also

- [Dokploy Deploy](../dokploy-deploy) - Deploy after migrations
- [Preview Deployment](../preview-deployment) - Test in preview environment
- [Update Provider](../update-provider) - Update application configuration
