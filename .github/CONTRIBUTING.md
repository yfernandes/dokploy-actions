# Contributing - Adding New Actions

This guide explains how to add a new GitHub Action to this repository.

## Quick Start

### 1. Create the Action Directory

```bash
mkdir -p .github/actions/my-action
```

### 2. Create `action.yml`

```yaml
# .github/actions/my-action/action.yml
name: 'My Action Name'
description: 'Clear, concise description of what this action does'
author: 'Your Name'

branding:
  icon: 'arrow-right'  # see https://feathericons.com/ for options
  color: 'blue'        # white, yellow, blue, green, red, orange, purple

inputs:
  my_input:
    description: 'Description of input'
    required: true
    default: 'default-value'

outputs:
  my_output:
    description: 'Description of output'
    value: ${{ steps.my_step.outputs.result }}

runs:
  using: 'composite'
  steps:
    - id: my_step
      run: |
        echo "result=success" >> $GITHUB_OUTPUT
      shell: bash
```

### 3. Create `README.md`

```markdown
# My Action

One-line description.

## Features

- ✅ Feature 1
- ✅ Feature 2

## Usage

### Basic Example

\`\`\`yaml
- uses: ./.github/actions/my-action
  with:
    my_input: 'value'
\`\`\`

### Marketplace Usage

\`\`\`yaml
- uses: yago/dokploy-actions/my-action@v1
  with:
    my_input: 'value'
\`\`\`

## Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `my_input` | Description | ✅ Yes |

## Outputs

| Output | Description |
|--------|-------------|
| `my_output` | Description |

## How It Works

1. Step 1
2. Step 2

## Examples

...

## Troubleshooting

...
```

### 4. Test Locally

```bash
# Install act (one-time)
# See: .github/TESTING_WITH_ACT.md

# Create a test workflow
cat > .github/workflows/test-my-action.yml << 'YAML'
name: Test My Action
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/my-action
        with:
          my_input: 'test-value'
YAML

# Run with act
act push
```

### 5. Commit & Push

```bash
git add .github/actions/my-action/
git commit -m "feat: add my-action"
git push origin main
```

### 6. Publish to Marketplace

Once your action is ready for release:

```bash
# Create a tag: <action-name>-v<version>
git tag my-action-v1.0.0
git push origin my-action-v1.0.0

# Create a release on GitHub
# → Releases → Draft new release
# → Select the tag → Publish
```

See [MARKETPLACE_GUIDE.md](./MARKETPLACE_GUIDE.md) for detailed instructions.

## Best Practices

✅ **DO:**
- Keep actions focused on a single responsibility
- Document inputs/outputs thoroughly with examples
- Test actions before publishing
- Use semantic versioning (MAJOR.MINOR.PATCH)
- Include error handling and clear error messages
- Add branding (icon + color) to `action.yml`

❌ **DON'T:**
- Create monolithic actions that do everything
- Skip documentation or examples
- Use generic names like "Deploy" or "Build"
- Break backward compatibility in patch versions
- Hardcode secrets (use GitHub Secrets instead)
- Publish without testing

## Action Types

### Composite Actions (Recommended)

Use shell scripts, combining existing tools:

```yaml
runs:
  using: 'composite'
  steps:
    - run: echo "Hello!"
      shell: bash
```

**Pros:** No dependencies, fast, portable
**Cons:** Limited to shell/script capabilities

### JavaScript Actions

Use Node.js for complex logic:

```yaml
runs:
  using: 'node20'
  main: 'index.js'
```

Requires `index.js` in the action directory.

### Docker Actions

Full control using Docker containers:

```yaml
runs:
  using: 'docker'
  image: 'docker://myimage:latest'
```

**Pros:** Any language, full environment control
**Cons:** Slower, larger, more complex

## File Structure Reference

```
.github/actions/my-action/
├── action.yml              # Action definition (required)
├── README.md               # Documentation (required)
├── index.js                # JavaScript implementation (if JS action)
├── Dockerfile              # Docker image (if Docker action)
├── entrypoint.sh           # Entrypoint script (if Docker action)
└── MARKETPLACE.md          # Marketplace metadata (optional)
```

## Naming Conventions

- **Directory name:** `kebab-case` (e.g., `my-action`)
- **Action name in YAML:** Title case (e.g., `My Action`)
- **Tags:** `{action-name}-v{version}` (e.g., `my-action-v1.0.0`)

## Versioning

Use semantic versioning:
- `MAJOR` - Breaking changes
- `MINOR` - New features (backward compatible)
- `PATCH` - Bug fixes

Example progression:
```
my-action-v1.0.0
my-action-v1.1.0  ← new feature
my-action-v1.1.1  ← bug fix
my-action-v2.0.0  ← breaking change
```

## Common Patterns

### Set Output

```yaml
- id: my_step
  run: |
    echo "result=success" >> $GITHUB_OUTPUT
  shell: bash
```

### Use Secrets

```yaml
- run: |
    API_KEY="${{ secrets.MY_SECRET }}"
  shell: bash
```

### Conditional Steps

```yaml
- if: github.ref == 'refs/heads/main'
  run: echo "On main branch"
  shell: bash
```

### Matrix Testing

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest, windows-latest]
runs-on: ${{ matrix.os }}
```

## Resources

- [Creating GitHub Actions](https://docs.github.com/en/actions/creating-actions)
- [Action Metadata Syntax](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions)
- [Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)
- [JavaScript Actions](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)
- [Docker Actions](https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action)

## Getting Help

1. Check existing actions in `.github/actions/` for examples
2. Review test workflows in `.github/workflows/`
3. Read GitHub Actions documentation
4. Open an issue if you have questions
