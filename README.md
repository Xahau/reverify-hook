# Hookstore Re-verify Hook

A GitHub Action to automatically re-verify your Xahau Hook on [Hookstore](https://hookstore.dev) when you publish a new release or tag.

## Features

- ✅ **Zero Configuration** - Automatically detects repository URL and source reference
- 📦 **Auto-publish** - Optionally create new releases automatically
- 🔐 **Secure** - Uses your Hookstore API key for authentication
- 🚀 **Multiple Triggers** - Works with tags, releases, or manual dispatch
- ⏳ **Wait for Completion** - Optionally block until verification completes (useful for CI/CD gates)

## Usage

### Basic Example

```yaml
name: Hookstore Re-verification

on:
  push:
    tags:
      - 'v*.*.*'
  release:
    types: [published]

jobs:
  reverify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Re-verify Hook
        uses: Xahau/reverify-hook@v1
        with:
          api_key: ${{ secrets.HOOKSTORE_API_KEY }}
```

### Wait for Completion

Block until verification completes (useful for CI/CD gates):

```yaml
- name: Re-verify and Wait
  uses: Xahau/reverify-hook@v1
  with:
    api_key: ${{ secrets.HOOKSTORE_API_KEY }}
    wait_for_completion: true
    timeout: 900  # 15 minutes
```

### Auto-create Release

Automatically create a new release if it doesn't exist:

```yaml
- name: Re-verify with Auto-create
  uses: Xahau/reverify-hook@v1
  with:
    api_key: ${{ secrets.HOOKSTORE_API_KEY }}
    create_if_missing: true
```

## Setup

### 1. Get Your Hookstore API Key

1. Go to [Hookstore Console](https://console.hookstore.dev/settings/api-keys)
2. Generate a new API key with `builds:trigger` permission
3. Copy the generated key (starts with `hks_`)

### 2. Add API Key to Repository Secrets

1. Go to your repository on GitHub
2. Navigate to **Settings → Secrets and variables → Actions**
3. Click **New repository secret**
4. Name: `HOOKSTORE_API_KEY`
5. Value: Your API key from step 1
6. Click **Add secret**

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api_key` | Yes | - | Your Hookstore API key |
| `api_url` | No | `https://api.hookstore.xahau.network` | API endpoint URL |
| `source_ref` | No | Auto-detected | Source reference (tag/branch/commit) |
| `create_if_missing` | No | `false` | Create new release if it doesn't exist |
| `wait_for_completion` | No | `false` | Block until verification completes |
| `timeout` | No | `600` | Max wait time in seconds (if waiting) |

## Outputs

| Output | Description |
|--------|-------------|
| `hook_slug` | The hook slug that was re-verified |
| `semver` | The semantic version that was re-verified |
| `release_id` | The release ID |
| `created` | Whether a new release was created |
| `status` | Verification status (queued, verified, failed, timeout) |

## How It Works

1. **Detects Context**: Automatically determines repository URL and source reference from GitHub context
2. **Finds Hook**: Looks up your hook by repository URL on Hookstore
3. **Finds/Creates Release**: Locates existing release or creates new one (if `create_if_missing: true`)
4. **Queues Verification**: Triggers re-verification on Hookstore
5. **Optionally Waits**: If `wait_for_completion: true`, polls status until verification completes

### Source Reference Detection

The action automatically detects the source reference in this order:

1. Manual `source_ref` input (if provided)
2. GitHub release tag name (if triggered by release event)
3. Git tag name (if triggered by tag push)
4. Manifest version from `hookstore.manifest.json` (if file exists)
5. Commit SHA (fallback)

## Examples

### Tag-based Release

```yaml
on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  reverify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Xahau/reverify-hook@v1
        with:
          api_key: ${{ secrets.HOOKSTORE_API_KEY }}
```

### GitHub Release

```yaml
on:
  release:
    types: [published]

jobs:
  reverify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Xahau/reverify-hook@v1
        with:
          api_key: ${{ secrets.HOOKSTORE_API_KEY }}
          create_if_missing: true
```

### CI/CD Gate

Block deployment until verification completes:

```yaml
jobs:
  verify-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Verify Hook
        id: verify
        uses: Xahau/reverify-hook@v1
        with:
          api_key: ${{ secrets.HOOKSTORE_API_KEY }}
          wait_for_completion: true
          timeout: 900
      
      - name: Deploy
        if: steps.verify.outputs.status == 'verified'
        run: |
          echo "Hook verified! Deploying..."
          # Your deployment steps here
```

## Versioning

This action uses semantic versioning. You can pin to:

- `@v1` - Latest v1.x.x (recommended)
- `@v1.0.0` - Specific version
- `@main` - Latest from main branch (not recommended for production)

## Troubleshooting

### "Hook not found for repository"

**Cause**: Your hook hasn't been published to Hookstore yet.

**Solution**: 
1. Publish your hook via the [Hookstore Console](https://console.hookstore.dev) first
2. Or use `create_if_missing: true` to automatically create the release

### "No release found for sourceRef"

**Cause**: No release exists for the specified source reference.

**Solution**: 
1. Create the release via the console first
2. Or use `create_if_missing: true` to automatically create it

### Verification Timeout

**Cause**: Verification took longer than the specified timeout.

**Solution**: 
1. Increase the `timeout` value (default: 600 seconds)
2. Check Hookstore Console for build errors
3. Verify your source code builds successfully

## Security

- ✅ API key stored as GitHub secret (encrypted)
- ✅ Builds run in isolated Docker containers on Hookstore
- ✅ Source code verified against on-chain hash
- ✅ All requests authenticated

**Best Practices**:
- Use a dedicated API key per repository
- Limit API key permissions to `builds:trigger` only
- Rotate API keys periodically
- Monitor API key usage in Console

## Support

- 📖 [Documentation](https://console.hookstore.dev/docs/github-actions)
- 💬 [Issues](https://github.com/Xahau/reverify-hook/issues)
- 🌐 [Hookstore Console](https://console.hookstore.dev)


