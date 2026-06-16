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
- **Grouping Strategy**: Updates are grouped by service category:
  - **VPN (gluetun)**: Updates every 4 weeks (critical service, less frequent updates) - labeled `vpn` and `critical`
  - **Arr Stack Downloaders**: prowlarr, radarr, sonarr, bazarr, sabnzbd, deluge
  - **Arr Stack Management**: tautulli, kometa
  - **Utility Services**: flaresolverr
  - **Website Stack**: organizr, ombi, nginx-proxy-manager, mariadb
  - **Misc Stack**: muse, scrutiny, threadfin

- **Update Schedule**: 
  - Most services: Monday mornings (before 3am UTC)
  - Gluetun (VPN): Every 4 weeks (conservative schedule for critical infrastructure)
- **Minimum Release Age**: Waits 3 days after a release before creating a PR (stability buffer)
- **Auto-merge**: Disabled by default (you review all PRs)
- **PR Limits**: 
  - Maximum 4 concurrent PRs
  - Maximum 3 new PRs created per run

### 4. GitHub Actions Validation

The `.github/workflows/validate-compose.yml` workflow:

- Automatically discovers and validates all `compose.yaml`, `compose.yml`, and `*compose*.yaml` files
- Checks Docker Compose syntax using `docker compose config --quiet`
- Runs on:
  - Pull requests with compose file changes
  - Pushes to main, master, and renovate branches
  - Any workflow file changes
- Provides fast feedback on Renovate PRs before you review

## How It Works

1. **Renovate Bot** runs periodically (based on schedule) and checks for new versions of your dependencies
2. **Creates PRs** with updated versions grouped by category
3. **GitHub Actions** automatically validates the compose files
4. **You review** each PR and decide whether to merge
5. **Merge** to update your services

## Manual Workflow Trigger

To manually check for updates:

1. Click the **"Dependency Dashboard"** issue in your GitHub repository (Renovate creates this)
2. Click on service updates to create PRs immediately
3. Or go to Renovate settings to reconfigure

## Customization

To modify the configuration, edit `renovate.json`:

- **Change update schedules**: Modify `packageRules[].schedule` (e.g., `"every 4 weeks"`, `"before 3am on Monday"`)
- **Adjust grouping**: Modify `packageRules` patterns to group different services
- **Enable auto-merge**: Set `"automerge": true` (currently disabled for all services)
- **Adjust release age**: Lower `minimumReleaseAge` for faster updates (currently 3 days)
- **Add service labels**: Add labels to rules (e.g., `"labels": ["critical"]`) to track important updates
- **Modify PR limits**: Adjust `prConcurrentLimit` (currently 4) and `prCreationLimit` (currently 3)

### Example: Adding a new critical service

```json
{
  "matchPackagePatterns": ["your-service-name"],
  "groupName": "Critical Service",
  "schedule": ["every 4 weeks"],
  "labels": ["critical", "review-required"]
}
```

## Disabling Renovate

If you want to disable Renovate:

1. Go to https://github.com/settings/installations
2. Find Renovate and click to configure
3. Remove this repository from the installation

Or add `"enabled": false` to `renovate.json`

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

### Conservative Gluetun Schedule
Gluetun is your VPN provider for the Arr Stack. Using a 4-week update cycle ensures:
- Stability of your VPN connection
- Time to test updates before deployment
- Reduced risk of breaking your entire download stack

### 3-Day Minimum Release Age
- Allows critical bugs to be discovered in new versions
- Prevents deploying unstable releases
- Still gets updates quickly after proven stability

### No Auto-merge
- All updates require your review
- Allows testing before merging
- Prevents unexpected service disruptions

### Dynamic Compose File Discovery
- GitHub Actions automatically finds all compose files
- Scales as your infrastructure grows
- Less maintenance if new stacks are added
- Uses `find` command for flexibility
