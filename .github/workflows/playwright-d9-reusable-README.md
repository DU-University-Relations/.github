# Playwright D9 Reusable Workflow

A reusable GitHub Actions workflow for running Playwright tests against Drupal 9 modules. This workflow sets up a complete Drupal 9 environment using DDEV, installs your module, and runs Playwright tests with configurable module enablement and test tags.

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

1. A Drupal module with a valid `.info.yml` file
2. Playwright tests in a `tests/` directory or compatible structure
3. Access to the d9-composer-managed repository via the `D9_PROFILE_PAT` secret

## Usage

### Basic Usage

```yaml
name: Playwright Tests

on:
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

### Matrix Strategy

Run different test suites in parallel:

```yaml
jobs:
  test-d9:
    strategy:
      matrix:
        include:
          - modules: 'du_banners'
            tags: '@banners'
          - modules: 'du_news'
            tags: '@news'
    uses: DU-University-Relations/.github/.github/workflows/playwright-d9-reusable.yml@main
    with:
      enabled_modules: ${{ matrix.modules }}
      test_tags: ${{ matrix.tags }}
    secrets:
      D9_PROFILE_PAT: ${{ secrets.D9_PROFILE_PAT }}
```

## Configuration

### Setting up D9_PROFILE_PAT Secret

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate a new token with `repo` scope
3. Add it as a repository or organization secret named `D9_PROFILE_PAT`

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

The workflow uploads a Playwright HTML report as `playwright-report-d9` which includes test results, screenshots, videos, and error traces. Download from the workflow run's Artifacts section.

## Related Resources

- [MS Teams Notification Action](../actions/ms-teams-notification/README.md)
- [Playwright D10 Reusable Workflow](./playwright-d10-reusable-README.md)
