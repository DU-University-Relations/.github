# MS Teams Notification Action

A reusable GitHub Action to send adaptive card notifications to Microsoft Teams webhooks. This action is designed specifically for pull request workflows and notifies teams about workflow failures or other important events.

## Features

- Sends rich adaptive card notifications to MS Teams
- Designed specifically for pull request workflows
- Automatically includes repository, branch, failure information, actor, and workflow details
- Shows the PR source branch
- Provides detailed failure information (optionally passed as input)
- Provides direct link to the workflow run
- Configurable title and webhook URL
- Easy to integrate into existing workflows

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `webhook-url` | MS Teams webhook URL | Yes | - |
| `title` | Notification title | No | `❌ Playwright Tests Failed` |
| `failure-info` | Optional failure information (job name, failed steps, etc.) | No | `Job: <job-name>` |

## Usage

### Basic Usage with Organization Secret

If your organization has the `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL` secret configured:

```yaml
- name: Notify MS Teams on failure
  if: failure()
  uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
  with:
    webhook-url: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
```

### Usage with Custom Webhook URL

If you want to use a different webhook URL (e.g., repository-specific):

```yaml
- name: Notify MS Teams on failure
  if: failure()
  uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
  with:
    webhook-url: ${{ secrets.CUSTOM_TEAMS_WEBHOOK }}
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
```

### Usage with Custom Failure Information

You can pass custom failure information to provide more context:

```yaml
- name: Run tests
  id: tests
  run: npm test
  continue-on-error: true

- name: Notify MS Teams on failure
  if: steps.tests.outcome == 'failure'
  uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
  with:
    webhook-url: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
    failure-info: "Tests failed in ${{ github.job }}"
```

### Advanced: Capturing Playwright Error Details

For Playwright tests, you can capture specific error messages using the `github` reporter and pass them to the notification:

```yaml
- name: Run Playwright Tests
  id: playwright
  shell: bash
  run: |
    # Use the github reporter and capture output with tee
    npx playwright test --reporter=github 2>&1 | tee test.log
    echo "exit_code=${PIPESTATUS[0]}" >> $GITHUB_OUTPUT
  continue-on-error: true

- name: Extract Failure Message
  id: extract
  if: steps.playwright.outputs.exit_code != '0'
  run: |
    # Extract the first error annotation from the log
    ERROR_MSG=$(grep "^::error" test.log | sed -E 's/^::error[^:]*:://' | head -n 1)
    
    # Fallback if no error found
    if [ -z "$ERROR_MSG" ]; then
      ERROR_MSG="Tests failed, but no specific error annotation was found."
    fi
    
    echo "Captured Error: $ERROR_MSG"
    
    # Write to output using multiline syntax
    echo "error_message<<EOF" >> $GITHUB_OUTPUT
    echo "$ERROR_MSG" >> $GITHUB_OUTPUT
    echo "EOF" >> $GITHUB_OUTPUT

- name: Notify MS Teams on failure
  if: steps.playwright.outputs.exit_code != '0'
  uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
  with:
    webhook-url: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
    failure-info: ${{ steps.extract.outputs.error_message }}
```

This approach:
- Uses Playwright's built-in `github` reporter which emits `::error` annotations
- Captures the test output with `tee` so it's both displayed and saved
- Extracts the first error message from the annotations
- Passes the specific error to the MS Teams notification
- **Note**: Requires `shell: bash` for `PIPESTATUS` support (as shown in the example)

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

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
```

## Workflow Conditions

The action has **no conditions built into it**. Users specify the condition when they use it, giving you full flexibility over when notifications are sent.

### Common Condition Examples

```yaml
# Only notify on failure
- name: Notify MS Teams on failure
  if: failure()
  uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
  with:
    webhook-url: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}

# Always notify regardless of success/failure
- name: Notify MS Teams
  if: always()
  uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
  with:
    webhook-url: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
    title: "Workflow Completed"

# Only notify on success
- name: Notify MS Teams on success
  if: success()
  uses: DU-University-Relations/.github/.github/actions/ms-teams-notification@main
  with:
    webhook-url: ${{ secrets.MS_TEAMS_WEBHOOK }}
    title: "✅ Tests Passed"
```

### Why This Approach is Best

- **Flexibility** - Users can decide when to trigger notifications based on their needs
- **Reusability** - Same action works for failures, successes, or always
- **Standard GitHub Actions pattern** - This is how most reusable actions work

The action itself just sends the notification whenever it runs. The calling workflow controls when it runs using the `if:` condition.

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

## Notification Information

The notification card includes the following information:

- **Repository**: The repository where the workflow is running (e.g., `DU-University-Relations/drupal-composer-managed`)
- **Branch**: The source branch of the pull request (e.g., `feature-branch`)
  - Note: This action is designed for pull request workflows and uses `github.head_ref`
- **Failure Info**: Information about what failed
  - If the `failure-info` input is provided, displays that custom message
  - Otherwise, defaults to showing the job name (e.g., `Job: playwright`)
  - Examples: `"Tests failed in playwright"`, `"Job: build"`, `"Step: Run Playwright tests failed"`
- **Triggered by**: The GitHub username that triggered the workflow
- **Workflow**: The name of the workflow that ran
- **View Workflow Run** button: Links directly to the workflow run logs on GitHub

## Notes

- **Designed for Pull Requests**: This action is specifically designed for pull request workflows and uses `github.head_ref` to display the PR source branch
- The `if: failure()` condition ensures the notification is only sent when the workflow fails
- You can use `if: always()` to send notifications regardless of workflow status
- Optionally pass custom failure information using the `failure-info` input for more detailed context
- The action uses curl to send the webhook request, which is available in all GitHub-hosted runners

## License

This action is maintained by DU University Relations.
