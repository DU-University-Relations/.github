# Playwright D10 Reusable Workflow

A reusable GitHub Actions workflow for running Playwright tests against Drupal 10 modules. This workflow sets up a complete Drupal 10 environment using DDEV, installs your module, and runs Playwright tests with configurable module enablement and test tags.

## Features

- Sets up a complete Drupal 10 environment using DDEV
- Automatically installs the drupal-composer-managed profile
- Copies your module into the test environment
- Enables specified Drupal modules for testing
- Runs Playwright tests with tag filtering
- Captures and reports test failures with detailed error messages
- Optionally sends MS Teams notifications on test failures
- Uploads Playwright HTML reports as artifacts
- Configurable profile reference (branch/tag/SHA)
- Uses standard `GITHUB_TOKEN` (no additional PAT required)

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `enabled_modules` | Space-separated list of modules to enable (e.g., "du_banners du_functional_testing") | Yes | - |
| `test_tags` | Playwright test tags to run (e.g., "@banners") | Yes | - |
| `profile_ref` | Optional ref (branch/tag/SHA) of drupal-composer-managed to use | No | `''` (default branch) |

## Secrets

| Secret | Description | Required |
|--------|-------------|----------|
| `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL` | MS Teams webhook URL for failure notifications | No |

**Note**: Unlike the D9 workflow, this workflow uses the standard `GITHUB_TOKEN` and does not require a separate PAT for accessing the drupal-composer-managed repository.

## Prerequisites

Your repository must contain:

1. **A Drupal module** with a valid `.info.yml` file
2. **Playwright tests** located in a `tests/` directory or compatible structure
3. **Public or organization access** to the drupal-composer-managed repository

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
  test-d10:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d10-reusable.yml@main
    with:
      enabled_modules: 'du_banners'
      test_tags: '@banners'
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
  test-d10:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d10-reusable.yml@main
    with:
      enabled_modules: 'du_banners'
      test_tags: '@banners'
    secrets:
      MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
```

### Testing Multiple Modules

Enable multiple modules by separating them with spaces:

```yaml
jobs:
  test-d10:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d10-reusable.yml@main
    with:
      enabled_modules: 'du_banners du_news du_events'
      test_tags: '@banners'
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
  test-d10:
    strategy:
      matrix:
        include:
          - modules: 'du_banners'
            tags: '@banners'
          - modules: 'du_news'
            tags: '@news'
          - modules: 'du_events'
            tags: '@events'
    uses: DU-University-Relations/.github/.github/workflows/playwright-d10-reusable.yml@main
    with:
      enabled_modules: ${{ matrix.modules }}
      test_tags: ${{ matrix.tags }}
    secrets:
      MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
```

### Using a Specific Profile Branch

Test against a specific branch, tag, or commit of drupal-composer-managed:

```yaml
jobs:
  test-d10:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d10-reusable.yml@main
    with:
      enabled_modules: 'du_banners'
      test_tags: '@banners'
      profile_ref: 'feature/new-theme'
```

### Testing Both D9 and D10

Run tests against both Drupal versions in a single workflow:

```yaml
name: Playwright Tests

on:
  push:
    branches: [main]
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

  test-d10:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d10-reusable.yml@main
    with:
      enabled_modules: 'du_banners'
      test_tags: '@banners'
    secrets:
      MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
```

## What This Workflow Does

1. **Checkout Module**: Checks out your module repository into the `module/` directory
2. **Checkout Drupal Upstream**: Clones the drupal-composer-managed repository using `GITHUB_TOKEN`
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
- **Base URL**: `https://drupal-composer-managed.ddev.site`
- **Drupal Profile**: `ducore`
- **Module Path**: `web/modules/packages/{module-name}`

## Artifacts

### Playwright Report

The workflow uploads a Playwright HTML report as an artifact named `playwright-report-d10`. This report is available even if tests fail and includes:

- Test results with screenshots
- Video recordings of test runs
- Detailed error traces
- Timeline views

To download:
1. Go to the workflow run in GitHub Actions
2. Scroll to the "Artifacts" section
3. Download `playwright-report-d10`
4. Extract and open `index.html` in a browser

## Differences from D9 Workflow

| Feature | D9 Workflow | D10 Workflow |
|---------|-------------|--------------|
| Upstream Repository | `d9-composer-managed` | `drupal-composer-managed` |
| Authentication | Requires `D9_PROFILE_PAT` | Uses standard `GITHUB_TOKEN` |
| Base URL | `d9-composer-managed.ddev.site` | `drupal-composer-managed.ddev.site` |
| Artifact Name | `playwright-report-d9` | `playwright-report-d10` |

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

### Access Denied to drupal-composer-managed

**Issue**: Cannot clone drupal-composer-managed repository
- **Cause**: Repository permissions or `GITHUB_TOKEN` scope issues
- **Solution**: Ensure the repository is accessible to your organization. Contact repository administrators if needed

### MS Teams Notification Not Sent

**Issue**: No notification received when tests fail
- **Cause**: `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL` secret not configured
- **Solution**: This is optional. Add the secret if you want notifications, or ignore this if not needed

## Migration from D9 to D10

If you're migrating from the D9 workflow to D10:

1. **Update the workflow reference**:
   ```yaml
   # Before
   uses: DU-University-Relations/.github/.github/workflows/playwright-d9-reusable.yml@main
   
   # After
   uses: DU-University-Relations/.github/.github/workflows/playwright-d10-reusable.yml@main
   ```

2. **Remove the D9_PROFILE_PAT secret**:
   - The D10 workflow uses `GITHUB_TOKEN` automatically
   - No need to pass any PAT secret

3. **Keep everything else the same**:
   - Inputs (`enabled_modules`, `test_tags`, `profile_ref`) remain identical
   - MS Teams notification secret name is the same
   - Test structure and tags don't need to change

## Notes

- **Module Naming**: The workflow automatically determines your module name from the repository name
- **Automatic Cleanup**: DDEV and test environments are automatically cleaned up after the workflow completes
- **Test Isolation**: Each workflow run gets a fresh Drupal installation
- **du_functional_testing**: This module is always enabled as it provides test utilities
- **Notifications are Optional**: The workflow checks if the MS Teams webhook secret exists before sending notifications
- **Error Messages**: The workflow captures the first error from failed tests and includes it in notifications
- **No Additional PAT Required**: Unlike D9, this workflow uses the standard `GITHUB_TOKEN` for repository access

## Related Resources

- [MS Teams Notification Action](../actions/ms-teams-notification/README.md)
- [Playwright D9 Reusable Workflow](./playwright-d9-reusable-README.md)
- [drupal-composer-managed Repository](https://github.com/DU-University-Relations/drupal-composer-managed)
- [Playwright Documentation](https://playwright.dev/)
- [DDEV Documentation](https://ddev.readthedocs.io/)

## License

This workflow is maintained by DU University Relations.
