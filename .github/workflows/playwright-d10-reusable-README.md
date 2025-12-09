# Playwright D10 Reusable Workflow

A reusable GitHub Actions workflow for running Playwright tests against Drupal 10 modules. This workflow sets up a complete Drupal 10 environment using DDEV, installs your module, and runs Playwright tests with configurable module enablement and test tags.

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

**Note**: This workflow uses the standard `GITHUB_TOKEN` and does not require a separate PAT.

## Prerequisites

Your repository must contain:

1. A Drupal module with a valid `.info.yml` file
2. Playwright tests in a `tests/` directory or compatible structure
3. Access to the drupal-composer-managed repository
4. Playwright configuration using Chromium browser (the workflow installs only Chromium to optimize CI performance)

## Usage

### Basic Usage

```yaml
name: Playwright Tests

on:
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

### Matrix Strategy

Run different test suites in parallel:

```yaml
jobs:
  test-d10:
    strategy:
      matrix:
        include:
          - modules: 'du_banners'
            tags: '@banners'
          - modules: 'du_news'
            tags: '@news'
    uses: DU-University-Relations/.github/.github/workflows/playwright-d10-reusable.yml@main
    with:
      enabled_modules: ${{ matrix.modules }}
      test_tags: ${{ matrix.tags }}
```

### Testing Both D9 and D10

```yaml
jobs:
  test-d9:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d9-reusable.yml@main
    with:
      enabled_modules: 'du_banners'
      test_tags: '@banners'
    secrets:
      D9_PROFILE_PAT: ${{ secrets.D9_PROFILE_PAT }}

  test-d10:
    uses: DU-University-Relations/.github/.github/workflows/playwright-d10-reusable.yml@main
    with:
      enabled_modules: 'du_banners'
      test_tags: '@banners'
```

## Configuration

### Setting up MS Teams Notifications (Optional)

Add the `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL` secret to your repository or organization settings. If not provided, notifications are skipped.

### Test Tags

Tags are defined in your test files and used to filter which tests to run:

```javascript
test('@banners should display banner correctly', async ({ page }) => {
  // test code
});
```

Use Playwright's grep syntax: `@banners`, `@banners|@news`, or `@banners @smoke`

## Artifacts

The workflow uploads a Playwright HTML report as `playwright-report-d10` which includes test results, screenshots, videos, and error traces. Download from the workflow run's Artifacts section.

## Differences from D9 Workflow

| Feature | D9 Workflow | D10 Workflow |
|---------|-------------|--------------|
| Upstream Repository | `d9-composer-managed` | `drupal-composer-managed` |
| Authentication | Requires `D9_PROFILE_PAT` | Uses standard `GITHUB_TOKEN` |
| Artifact Name | `playwright-report-d9` | `playwright-report-d10` |

## Related Resources

- [MS Teams Notification Action](../actions/ms-teams-notification/README.md)
- [Playwright D9 Reusable Workflow](./playwright-d9-reusable-README.md)
