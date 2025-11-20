# MS Teams Notification Action

A reusable GitHub Action to send adaptive card notifications to Microsoft Teams webhooks. This action is particularly useful for notifying teams about workflow failures or other important events.

## Features

- Sends rich adaptive card notifications to MS Teams
- Includes repository, branch, commit, actor, and workflow information
- Provides direct link to the failed workflow run
- Configurable title and webhook URL
- Easy to integrate into existing workflows

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `webhook-url` | MS Teams webhook URL | Yes | - |
| `title` | Notification title | No | `❌ Playwright Tests Failed` |
| `github-repository` | GitHub repository name | Yes | - |
| `github-ref-name` | Branch/ref name | Yes | - |
| `github-sha` | Commit SHA | Yes | - |
| `github-actor` | User who triggered the workflow | Yes | - |
| `github-workflow` | Workflow name | Yes | - |
| `github-server-url` | GitHub server URL | Yes | - |
| `github-run-id` | Workflow run ID | Yes | - |

## Usage

### Basic Usage with Organization Secret

If your organization has the `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL` secret configured:

```yaml
- name: Notify MS Teams on failure
  if: failure()
  uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
  with:
    webhook-url: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
    github-repository: ${{ github.repository }}
    github-ref-name: ${{ github.ref_name }}
    github-sha: ${{ github.sha }}
    github-actor: ${{ github.actor }}
    github-workflow: ${{ github.workflow }}
    github-server-url: ${{ github.server_url }}
    github-run-id: ${{ github.run_id }}
```

### Usage with Custom Webhook URL

If you want to use a different webhook URL (e.g., repository-specific):

```yaml
- name: Notify MS Teams on failure
  if: failure()
  uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
  with:
    webhook-url: ${{ secrets.CUSTOM_TEAMS_WEBHOOK }}
    github-repository: ${{ github.repository }}
    github-ref-name: ${{ github.ref_name }}
    github-sha: ${{ github.sha }}
    github-actor: ${{ github.actor }}
    github-workflow: ${{ github.workflow }}
    github-server-url: ${{ github.server_url }}
    github-run-id: ${{ github.run_id }}
```

### Usage with Custom Title

Customize the notification title for different scenarios:

```yaml
- name: Notify MS Teams on build failure
  if: failure()
  uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
  with:
    webhook-url: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
    title: "🔨 Build Failed"
    github-repository: ${{ github.repository }}
    github-ref-name: ${{ github.ref_name }}
    github-sha: ${{ github.sha }}
    github-actor: ${{ github.actor }}
    github-workflow: ${{ github.workflow }}
    github-server-url: ${{ github.server_url }}
    github-run-id: ${{ github.run_id }}
```

### Complete Workflow Example

Here's a complete example showing how to integrate this action into your workflow:

```yaml
name: Playwright Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run Playwright tests
        run: npx playwright test

      - name: Notify MS Teams on failure
        if: failure()
        uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
        with:
          webhook-url: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
          github-repository: ${{ github.repository }}
          github-ref-name: ${{ github.ref_name }}
          github-sha: ${{ github.sha }}
          github-actor: ${{ github.actor }}
          github-workflow: ${{ github.workflow }}
          github-server-url: ${{ github.server_url }}
          github-run-id: ${{ github.run_id }}

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
```

## Configuration

### Setting up the Webhook URL Secret

The webhook URL should be stored as a secret in your repository or organization:

1. **Organization-level secret** (recommended for shared use):
   - Go to your organization settings
   - Navigate to Secrets and variables → Actions
   - Create a new secret named `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL`
   - Paste your MS Teams webhook URL

2. **Repository-level secret** (for repository-specific webhooks):
   - Go to your repository settings
   - Navigate to Secrets and variables → Actions
   - Create a new secret with your webhook URL

### Getting a MS Teams Webhook URL

1. In MS Teams, go to the channel where you want notifications
2. Click the three dots (•••) next to the channel name
3. Select "Connectors" or "Workflows"
4. Search for "Incoming Webhook"
5. Configure the webhook and copy the URL

## Notes

- The `if: failure()` condition ensures the notification is only sent when the workflow fails
- You can use `if: always()` to send notifications regardless of workflow status
- All GitHub context inputs are required to provide complete information in the notification
- The action uses curl to send the webhook request, which is available in all GitHub-hosted runners

## License

This action is maintained by DU University Relations.
