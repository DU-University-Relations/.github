# Pantheon Sync Reusable Workflow

A reusable GitHub Actions workflow for syncing code between GitHub and Pantheon. This workflow handles two-way synchronization:

- **Push to Pantheon**: When code is pushed to GitHub, it is automatically deployed to the Pantheon Git repository.
- **Mirror from Pantheon**: On a schedule or manual trigger, changes from Pantheon are pulled back into GitHub.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `git_user_email` | Git user email for commits | No | `google@du.edu` |
| `git_user_name` | Git user name for commits | No | `DU-Automation` |
| `branch` | Branch to sync between GitHub and Pantheon | No | `master` |

## Secrets

| Secret | Description | Required |
|--------|-------------|----------|
| `PANTHEON_SSH_KEY` | SSH private key for Pantheon access | Yes |
| `SSH_CONFIG` | SSH config for Pantheon host | Yes |
| `KNOWN_HOSTS` | SSH known hosts for Pantheon | Yes |
| `PANTHEON_REPO` | Pantheon Git repository URL (SSH) | Yes |

## Prerequisites

Your repository must have the following secrets configured:

1. **`PANTHEON_SSH_KEY`** — An SSH private key authorized to access your Pantheon site's Git repository
2. **`SSH_CONFIG`** — SSH configuration for the Pantheon host (e.g., `Host codeserver.dev.*` entries)
3. **`KNOWN_HOSTS`** — SSH known hosts entry for the Pantheon Git server
4. **`PANTHEON_REPO`** — The SSH URL of your Pantheon site's Git repository (found in the Pantheon dashboard under "Connection Info")

## Usage

### Basic Usage

Create a workflow file in your site repository (e.g., `.github/workflows/pantheon_sync.yml`):

```yaml
name: Pantheon Sync

on:
  push:
    branches:
      - 'master'
  schedule:
    - cron: '0 3 * * *'
  workflow_dispatch:

jobs:
  sync:
    uses: DU-University-Relations/.github/.github/workflows/pantheon-sync-reusable.yml@main
    secrets:
      PANTHEON_SSH_KEY: ${{ secrets.PANTHEON_SSH_KEY }}
      SSH_CONFIG: ${{ secrets.SSH_CONFIG }}
      KNOWN_HOSTS: ${{ secrets.KNOWN_HOSTS }}
      PANTHEON_REPO: ${{ secrets.PANTHEON_REPO }}
```

### Custom Git Identity

Use a different Git user for commits:

```yaml
jobs:
  sync:
    uses: DU-University-Relations/.github/.github/workflows/pantheon-sync-reusable.yml@main
    with:
      git_user_email: 'ci-bot@example.com'
      git_user_name: 'CI Bot'
    secrets:
      PANTHEON_SSH_KEY: ${{ secrets.PANTHEON_SSH_KEY }}
      SSH_CONFIG: ${{ secrets.SSH_CONFIG }}
      KNOWN_HOSTS: ${{ secrets.KNOWN_HOSTS }}
      PANTHEON_REPO: ${{ secrets.PANTHEON_REPO }}
```

### Custom Branch

Sync a branch other than `master`:

```yaml
jobs:
  sync:
    uses: DU-University-Relations/.github/.github/workflows/pantheon-sync-reusable.yml@main
    with:
      branch: 'main'
    secrets:
      PANTHEON_SSH_KEY: ${{ secrets.PANTHEON_SSH_KEY }}
      SSH_CONFIG: ${{ secrets.SSH_CONFIG }}
      KNOWN_HOSTS: ${{ secrets.KNOWN_HOSTS }}
      PANTHEON_REPO: ${{ secrets.PANTHEON_REPO }}
```

## How It Works

### Push to Pantheon (on `push` event)

1. **Checkout**: The workflow checks out the repository code
2. **SSH Setup**: Configures SSH keys and known hosts for Pantheon access
3. **Pull**: Pulls latest changes from Pantheon to avoid conflicts (allows unrelated histories)
4. **Push**: Pushes the merged code to the Pantheon Git repository

### Mirror from Pantheon (on `schedule` or `workflow_dispatch` event)

1. **Checkout**: The workflow checks out the repository code
2. **SSH Setup**: Configures SSH keys and known hosts for Pantheon access
3. **Pull**: Rebases local code on top of Pantheon changes
4. **Commit**: Commits any new changes from Pantheon
5. **Push**: Force-pushes the updated code back to GitHub

## Setting Up Pantheon Secrets

### Finding Your Pantheon Repository URL

1. Log in to the [Pantheon Dashboard](https://dashboard.pantheon.io/)
2. Select your site
3. Go to the **Dev** environment
4. Click **Connection Info**
5. Copy the **Git Clone Command** — the URL is the SSH repository URL (e.g., `ssh://codeserver.dev.xxx@codeserver.dev.xxx.drush.in:2222/~/repository.git`)

### Generating an SSH Key

```bash
ssh-keygen -t ed25519 -C "pantheon-deploy" -f pantheon_key -N ""
```

Add the public key (`pantheon_key.pub`) to your Pantheon account under **SSH Keys**, and store the private key (`pantheon_key`) as the `PANTHEON_SSH_KEY` secret.

### SSH Config Example

```
Host *.drush.in
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  LogLevel QUIET
```

## Related Resources

- [PHPUnit D10 Reusable Workflow](./phpunit-d10-reusable-README.md)
- [Playwright D10 Reusable Workflow](./playwright-d10-reusable-README.md)
