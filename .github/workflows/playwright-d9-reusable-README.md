# Playwright D9 Reusable Workflow

A reusable GitHub Actions workflow for running Playwright tests against Drupal 9 modules. This workflow sets up a complete Drupal 9 environment using DDEV, installs your module, and runs Playwright tests with configurable module enablement and test tags.

## Features

- Sets up a complete Drupal 9 environment using DDEV
- Automatically installs the d9-composer-managed profile
- Copies your module into the test environment
- Enables specified Drupal modules for testing
- Runs Playwright tests with tag filtering
- Captures and reports test failures with detailed error messages
- Optionally sends MS Teams notifications on test failures
- Uploads Playwright HTML reports as artifacts
- Configurable profile reference (branch/tag/SHA)

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `enabled_modules` | Space-separated list of modules to enable (e.g., "du_banners du_functional_testing") | Yes | - |
| `test_tags` | Playwright test tags to run (e.g., "@banners") | Yes | - |
| `profile_ref` | Optional ref (branch/tag/SHA) of d9-composer-managed to use | No | `''` (default branch) |

## Secrets

| Secret | Description | Required |
|--------|-------------|----------|
| `D9_PROFILE_PAT` | Personal access token for d9-composer-managed repository | Yes |
| `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL` | MS Teams webhook URL for failure notifications | No |

## Prerequisites

Your repository must contain:

1. **A Drupal module** with a valid `.info.yml` file
2. **Playwright tests** located in a `tests/` directory or compatible structure
3. **Access to the d9-composer-managed repository** via the `D9_PROFILE_PAT` secret

## Usage

### Basic Usage

Create a workflow file (e.g., `.github/workflows/playwright.yml`) in your module repository:

```yaml
name: Playwright Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test-d9:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d9-reusable.yml@main
    with:
      enabled_modules: 'du_banners'
      test_tags: '@banners'
    secrets:
      D9_PROFILE_PAT: ${{ secrets.D9_PROFILE_PAT }}
```

### Usage with MS Teams Notifications

Enable failure notifications by providing the webhook URL secret:

```yaml
name: Playwright Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test-d9:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d9-reusable.yml@main
    with:
      enabled_modules: 'du_banners'
      test_tags: '@banners'
    secrets:
      D9_PROFILE_PAT: ${{ secrets.D9_PROFILE_PAT }}
      MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
```

### Testing Multiple Modules

Enable multiple modules by separating them with spaces:

```yaml
jobs:
  test-d9:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d9-reusable.yml@main
    with:
      enabled_modules: 'du_banners du_news du_events'
      test_tags: '@banners'
    secrets:
      D9_PROFILE_PAT: ${{ secrets.D9_PROFILE_PAT }}
```

### Running Multiple Test Suites

Use a matrix strategy to run different test suites in parallel:

```yaml
name: Playwright Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test-d9:
    strategy:
      matrix:
        include:
          - modules: 'du_banners'
            tags: '@banners'
          - modules: 'du_news'
            tags: '@news'
          - modules: 'du_events'
            tags: '@events'
    uses: DU-University-Relations/.github/.github/workflows/playwright-d9-reusable.yml@main
    with:
      enabled_modules: ${{ matrix.modules }}
      test_tags: ${{ matrix.tags }}
    secrets:
      D9_PROFILE_PAT: ${{ secrets.D9_PROFILE_PAT }}
      MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
```

### Using a Specific Profile Branch

Test against a specific branch, tag, or commit of d9-composer-managed:

```yaml
jobs:
  test-d9:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d9-reusable.yml@main
    with:
      enabled_modules: 'du_banners'
      test_tags: '@banners'
      profile_ref: 'feature/new-theme'
    secrets:
      D9_PROFILE_PAT: ${{ secrets.D9_PROFILE_PAT }}
```

## What This Workflow Does

1. **Checkout Module**: Checks out your module repository into the `module/` directory
2. **Checkout Drupal Upstream**: Clones the d9-composer-managed repository using the provided PAT
3. **Setup DDEV**: Installs and configures DDEV for Drupal development
4. **Setup Node**: Installs Node.js 22 for running Playwright
5. **Copy Module**: Copies your module to `web/modules/packages/{module-name}` in the upstream
6. **Install Dependencies**: Runs `composer install` to get all PHP dependencies
7. **Install Drupal**: Installs Drupal using the `ducore` profile via `drush`
8. **Enable Modules**: Enables `du_functional_testing` and your specified modules
9. **Install Test Dependencies**: Runs `npm install` and generates test roles
10. **Install Playwright Browsers**: Downloads and installs required browser binaries
11. **Run Playwright Tests**: Executes tests matching your specified tags
12. **Extract Failure Messages**: Captures detailed error messages from test failures
13. **Send Notifications**: Optionally sends MS Teams notification if tests fail and webhook is configured
14. **Upload Reports**: Uploads the Playwright HTML report as a workflow artifact

## Configuration

### Setting up Secrets

#### D9_PROFILE_PAT (Required)

This Personal Access Token must have access to the private `DU-University-Relations/d9-composer-managed` repository.

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate a new token with `repo` scope
3. Add it as a repository or organization secret named `D9_PROFILE_PAT`

#### MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL (Optional)

This webhook URL is used to send failure notifications to MS Teams. If not provided, notifications are simply skipped.

1. **Organization-level secret** (recommended):
   - Go to your organization settings
   - Navigate to Secrets and variables → Actions
   - Create a new secret named `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL`
   - Paste your MS Teams webhook URL

2. **Repository-level secret**:
   - Go to your repository settings
   - Navigate to Secrets and variables → Actions
   - Create a new secret named `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL`
   - Paste your MS Teams webhook URL

### Test Tags

Playwright test tags are used to filter which tests to run. Tags are typically defined in your test files:

```javascript
test('@banners should display banner correctly', async ({ page }) => {
  // test code
});
```

You can specify multiple tags using Playwright's grep syntax:
- `@banners` - runs tests tagged with @banners
- `@banners|@news` - runs tests tagged with @banners OR @news
- `@banners @smoke` - runs tests tagged with BOTH @banners AND @smoke

## Environment

The workflow runs on:
- **Runner**: `ubuntu-22.04`
- **Node Version**: 22
- **Base URL**: `https://d9-composer-managed.ddev.site`
- **Drupal Profile**: `ducore`
- **Module Path**: `web/modules/packages/{module-name}`

## Artifacts

### Playwright Report

The workflow uploads a Playwright HTML report as an artifact named `playwright-report-d9`. This report is available even if tests fail and includes:

- Test results with screenshots
- Video recordings of test runs
- Detailed error traces
- Timeline views

To download:
1. Go to the workflow run in GitHub Actions
2. Scroll to the "Artifacts" section
3. Download `playwright-report-d9`
4. Extract and open `index.html` in a browser

## Troubleshooting

### Tests Not Running

**Issue**: No tests are executed
- **Cause**: Test tags don't match any tests in your repository
- **Solution**: Verify your test tags match the `@tag` annotations in your test files

### Module Not Found

**Issue**: Module cannot be enabled
- **Cause**: Module name doesn't match repository name
- **Solution**: The workflow uses the repository name as the module name. Ensure your `.info.yml` file matches your repository name

### DDEV Errors

**Issue**: DDEV setup fails
- **Cause**: Resource constraints or DDEV compatibility issues
- **Solution**: This is rare but may occur on GitHub's runners. Re-running the workflow usually resolves it

### Access Denied to d9-composer-managed

**Issue**: Cannot clone d9-composer-managed repository
- **Cause**: Invalid or expired `D9_PROFILE_PAT`
- **Solution**: Generate a new Personal Access Token with `repo` scope and update the secret

### MS Teams Notification Not Sent

**Issue**: No notification received when tests fail
- **Cause**: `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL` secret not configured
- **Solution**: This is optional. Add the secret if you want notifications, or ignore this if not needed

## Notes

- **Module Naming**: The workflow automatically determines your module name from the repository name
- **Automatic Cleanup**: DDEV and test environments are automatically cleaned up after the workflow completes
- **Test Isolation**: Each workflow run gets a fresh Drupal installation
- **du_functional_testing**: This module is always enabled as it provides test utilities
- **Notifications are Optional**: The workflow checks if the MS Teams webhook secret exists before sending notifications
- **Error Messages**: The workflow captures the first error from failed tests and includes it in notifications

## Related Resources

- [MS Teams Notification Action](../actions/ms-teams-notification/README.md)
- [d9-composer-managed Repository](https://github.com/DU-University-Relations/d9-composer-managed)
- [Playwright Documentation](https://playwright.dev/)
- [DDEV Documentation](https://ddev.readthedocs.io/)

## License

This workflow is maintained by DU University Relations.
