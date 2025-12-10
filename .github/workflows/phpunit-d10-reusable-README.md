# PHPUnit D10 Reusable Workflow

A reusable GitHub Actions workflow for running PHPUnit tests against Drupal 10 modules. This workflow sets up a complete Drupal 10 environment using DDEV, installs your module, and runs PHPUnit tests with configurable test paths and PHPUnit configuration.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `test_paths` | Space-separated list of paths to test (e.g., "web/profiles/custom web/modules/custom") | Yes | - |
| `phpunit_config` | Path to PHPUnit configuration file (e.g., "web/core/phpunit.xml.dist") | No | `web/core/phpunit.xml.dist` |
| `profile_ref` | Optional ref (branch/tag/SHA) of drupal-composer-managed to use | No | `''` (default branch) |

## Secrets

| Secret | Description | Required |
|--------|-------------|----------|
| `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL` | MS Teams webhook URL for failure notifications | No |

## Prerequisites

Your repository must contain:

1. A Drupal module with a valid `.info.yml` file
2. PHPUnit tests in appropriate directories (e.g., `tests/src/Unit/`, `tests/src/Kernel/`, `tests/src/Functional/`)
3. The workflow uses `GITHUB_TOKEN` for accessing the drupal-composer-managed repository (no additional PAT required)

## Usage

### Basic Usage

Test custom modules and profiles:

```yaml
name: PHPUnit Tests

on:
  pull_request:
    branches: [main]

jobs:
  test-d10:
    uses: DU-University-Relations/.github/.github/workflows/phpunit-d10-reusable.yml@main
    with:
      test_paths: 'web/profiles/custom web/modules/custom'
    secrets:
      MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
```

### Custom PHPUnit Configuration

Use a custom PHPUnit configuration file:

```yaml
jobs:
  test-d10:
    uses: DU-University-Relations/.github/.github/workflows/phpunit-d10-reusable.yml@main
    with:
      test_paths: 'web/modules/custom'
      phpunit_config: 'web/core/phpunit.custom.xml'
```

### Using a Specific Upstream Branch

Test against a specific version of the upstream:

```yaml
jobs:
  test-d10:
    uses: DU-University-Relations/.github/.github/workflows/phpunit-d10-reusable.yml@main
    with:
      test_paths: 'web/modules/custom'
      profile_ref: 'develop'
```

## Configuration

### Setting up MS Teams Notifications (Optional)

Add the `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL` secret to your repository or organization settings. If not provided, notifications are skipped.

### Test Paths

The `test_paths` input should point to directories containing PHPUnit tests. Common patterns:

- `web/modules/custom` - Test all custom modules
- `web/profiles/custom` - Test custom profiles
- `web/modules/packages/du_example_module` - Test a specific module
- `web/profiles/custom web/modules/custom` - Test multiple paths

## How It Works

1. **Checkout**: The workflow checks out your module into a `module` directory and the drupal-composer-managed upstream into an `upstream` directory
2. **Copy**: Your module is copied to `upstream/web/modules/packages/[module-name]`
3. **Setup**: DDEV is configured and Composer dependencies are installed with caching
4. **Validation**: composer.json and composer.lock are validated
5. **Test**: PHPUnit tests are executed from the specified paths (PHPUnit handles its own database and Drupal bootstrap as needed)
6. **Notify**: On failure, an optional MS Teams notification is sent

## Related Resources

- [MS Teams Notification Action](../actions/ms-teams-notification/README.md)
- [PHPUnit D9 Reusable Workflow](./phpunit-d9-reusable-README.md)
- [Playwright D10 Reusable Workflow](./playwright-d10-reusable-README.md)
