# Renovate Bot Configuration

This repository is configured to use [Renovate](https://www.renovatebot.com/) for automated dependency updates.

## Setup Instructions

### 1. Install Renovate Bot on GitHub

1. Go to https://github.com/apps/renovate
2. Click "Install" and select your repository
3. Grant the necessary permissions
4. Renovate will automatically create a setup PR within 1 hour

### 2. Merge the Setup PR

- Renovate will create an initial PR with the configuration
- Review and merge it (default configuration is included in `renovate.json`)

### 3. Configuration Details

The current `renovate.json` includes:

- **Docker Image Updates**: Detects and updates all Docker image versions in `compose.yaml` files
- **Grouping Strategy**: Updates are grouped by service category using package name matching:
  - **VPN (gluetun)**: Matches `/gluetun/` - labeled `vpn` and `critical` - updates at 3am on Monday
  - **Arr Stack Downloaders**: Matches prowlarr, radarr, sonarr, bazarr, sabnzbd, deluge
  - **Arr Stack Management**: Matches tautulli, kometa
  - **Utility Services**: Matches flaresolverr
  - **Website Stack**: Matches organizr, ombi, nginx-proxy-manager, mariadb
  - **Misc Stack**: Matches muse, scrutiny, threadfin

- **Update Schedule**: 
  - Most Docker images: Before 3am on Monday (UTC)
  - Gluetun (VPN): Runs at 3am on Monday separately (critical service)
- **Minimum Release Age**: Waits 3 days after a release before creating a PR (stability buffer)
- **Auto-merge**: Disabled by default (all PRs require manual review)
- **PR Limits**: 
  - Maximum 5 concurrent PRs
  - Semantic commit messages with `chore(deps):` prefix

### 4. GitHub Actions Workflows

**Renovate Workflow** (`.github/workflows/renovate.yml`):
- Runs on a schedule: Every Sunday at 4 AM UTC
- Can be manually triggered via `workflow_dispatch`
- Scans only the `homelab-stacks` repository (via `RENOVATE_REPOSITORIES` env var)
- Uses the configuration from `renovate.json`
- Creates PRs for dependency updates based on your schedule

**Validation Workflow** (`.github/workflows/validate-compose.yml`):
- Automatically discovers and validates all `compose.yaml`, `compose.yml`, and `*compose*.yaml` files
- Checks Docker Compose syntax using `docker compose config --quiet`
- Runs on:
  - Pull requests with compose file changes
  - Pushes to main, master, and renovate branches
  - Any GitHub Actions workflow file changes
- Provides fast feedback on Renovate PRs before you review them

## How It Works

1. **GitHub Actions** runs the Renovate workflow on schedule (every Sunday at 4am UTC)
2. **Renovate Bot** scans the `homelab-stacks` repository for dependency updates
3. **Creates PRs** with updated versions grouped by category from your `renovate.json`
4. **Validation Workflow** automatically checks all compose files in each PR
5. **You review** each PR and decide whether to merge
6. **Merge** to update your services in production

## Manual Workflow Trigger

To manually run Renovate checks immediately (instead of waiting for the scheduled Sunday run):

1. Go to your repository on GitHub
2. Click the **Actions** tab
3. Find **"Renovate"** workflow on the left
4. Click **"Run workflow"** → **"Run workflow"** button
5. The workflow will execute and create PRs if updates are available

To check for updates manually without running the workflow:
1. Look for the **"Dependency Dashboard"** issue in your GitHub Issues
2. Renovate creates this automatically
3. Click on it to see all available updates and create PRs manually

## Customization

To modify the configuration, edit `renovate.json`:

- **Change update schedules**: Modify `packageRules[].schedule` (e.g., `"before 3am on Monday"`, `"at 3am on Monday"`)
- **Adjust grouping**: Modify `packageRules[].matchPackageNames` regex patterns to group different services
- **Enable auto-merge**: Set `"automerge": true` (currently disabled for all services)
- **Adjust release age**: Lower `minimumReleaseAge` for faster updates (currently 3 days)
- **Add service labels**: Add labels to rules (e.g., `"labels": ["critical"]`) to track important updates
- **Modify PR limits**: Adjust `prConcurrentLimit` (currently 5)

### Example: Adding a new critical service

```json
{
  "groupName": "My Critical Service",
  "groupSlug": "my-critical-service",
  "schedule": ["at 3am on Monday"],
  "labels": ["critical", "review-required"],
  "matchPackageNames": ["/my-service-name/"]
}
```

**Note**: Package name matching uses regex patterns enclosed in forward slashes (e.g., `/service-name/` or `/service1|service2/`)

## Disabling Renovate

If you want to disable Renovate:

1. Go to https://github.com/settings/installations
2. Find Renovate and click to configure
3. Remove this repository from the installation

Or add `"enabled": false` to `renovate.json`

## Configuration Migration

When Renovate runs, it automatically migrates your configuration to ensure compatibility:
- Converts `matchPackagePatterns` → `matchPackageNames` with regex format
- Validates all settings against the latest schema
- Shows warnings about deprecated options (if any)

This is normal and expected—no action needed on your part. The migrated config is used for the run but your original `renovate.json` remains unchanged.

## Security Considerations

- Review all PRs before merging
- Test updates in a staging environment if possible
- Pay attention to major version updates (may require configuration changes)
- Security vulnerabilities are flagged separately and labeled `security`

## Supported Registries

Renovate automatically detects images from:

- Docker Hub
- GitHub Container Registry (ghcr.io)
- Quay.io
- Private registries (with additional setup)

## Why These Settings?

### Separate Gluetun Schedule
Gluetun is your critical VPN provider for the Arr Stack. Running it on the same schedule (`at 3am on Monday`) but in its own package rule ensures:
- Updates are reviewed separately from other services
- Labeled as `vpn` and `critical` for visibility
- Easier to test VPN changes independently
- Reduces risk of accidentally merging critical infrastructure updates with other changes

### Monday Morning Updates
- Gives you the work week to monitor and react to any issues
- Scheduled before business hours (3am UTC) to minimize service disruption
- All other updates run before the critical VPN updates

### 3-Day Minimum Release Age
- Allows critical bugs to be discovered in new versions
- Prevents deploying unstable releases
- Still gets updates quickly after proven stability

### No Auto-merge
- All updates require your manual review
- Allows testing before merging
- Prevents unexpected service disruptions

### Dynamic Compose File Discovery
- GitHub Actions automatically finds all compose files in the repo
- Scales as your infrastructure grows
- Less maintenance if new stacks are added
- Uses `find` command for flexibility

### Repository Restriction
- `RENOVATE_REPOSITORIES` environment variable restricts scanning to only `homelab-stacks`
- Prevents Renovate from scanning all your other repositories
- Keeps updates isolated to this infrastructure
