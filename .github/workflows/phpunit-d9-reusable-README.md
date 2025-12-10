# PHPUnit D9 Reusable Workflow

A reusable GitHub Actions workflow for running PHPUnit tests against Drupal 9 modules. This workflow sets up a complete Drupal 9 environment using DDEV, installs your module, and runs PHPUnit tests with configurable test paths and PHPUnit configuration.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `test_paths` | Space-separated list of paths to test (e.g., "web/profiles/custom web/modules/custom") | Yes | - |
| `phpunit_config` | Path to PHPUnit configuration file (e.g., "web/core/phpunit.xml.dist") | No | `web/core/phpunit.xml.dist` |
| `enabled_modules` | Space-separated list of modules to enable before running tests (e.g., "du_example_module") | No | `''` |
| `profile_ref` | Optional ref (branch/tag/SHA) of d9-composer-managed to use | No | `''` (default branch) |

## Secrets

| Secret | Description | Required |
|--------|-------------|----------|
| `D9_PROFILE_PAT` | Personal access token for d9-composer-managed repository | Yes |
| `MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL` | MS Teams webhook URL for failure notifications | No |

## Prerequisites

Your repository must contain:

1. A Drupal module with a valid `.info.yml` file
2. PHPUnit tests in appropriate directories (e.g., `tests/src/Unit/`, `tests/src/Kernel/`, `tests/src/Functional/`)
3. Access to the d9-composer-managed repository via the `D9_PROFILE_PAT` secret

## Usage

### Basic Usage

Test custom modules and profiles:

```yaml
name: PHPUnit Tests

on:
  pull_request:
    branches: [main]

jobs:
  test-d9:
    uses: DU-University-Relations/.github/.github/workflows/phpunit-d9-reusable.yml@main
    with:
      test_paths: 'web/profiles/custom web/modules/custom'
    secrets:
      D9_PROFILE_PAT: ${{ secrets.D9_PROFILE_PAT }}
      MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL: ${{ secrets.MS_TEAMS_FAILED_TEST_RUN_WEBHOOK_URL }}
```

### With Module Enablement

Enable specific modules before running tests:

```yaml
jobs:
  test-d9:
    uses: DU-University-Relations/.github/.github/workflows/phpunit-d9-reusable.yml@main
    with:
      test_paths: 'web/modules/packages/du_example_module'
      enabled_modules: 'du_example_module du_dependency_module'
    secrets:
      D9_PROFILE_PAT: ${{ secrets.D9_PROFILE_PAT }}
```

### Custom PHPUnit Configuration

Use a custom PHPUnit configuration file:

```yaml
jobs:
  test-d9:
    uses: DU-University-Relations/.github/.github/workflows/phpunit-d9-reusable.yml@main
    with:
      test_paths: 'web/modules/custom'
      phpunit_config: 'web/core/phpunit.custom.xml'
    secrets:
      D9_PROFILE_PAT: ${{ secrets.D9_PROFILE_PAT }}
```

### Using a Specific Upstream Branch

Test against a specific version of the upstream:

```yaml
jobs:
  test-d9:
    uses: DU-University-Relations/.github/.github/workflows/phpunit-d9-reusable.yml@main
    with:
      test_paths: 'web/modules/custom'
      profile_ref: 'develop'
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

### Test Paths

The `test_paths` input should point to directories containing PHPUnit tests. Common patterns:

- `web/modules/custom` - Test all custom modules
- `web/profiles/custom` - Test custom profiles
- `web/modules/packages/du_example_module` - Test a specific module
- `web/profiles/custom web/modules/custom` - Test multiple paths

## How It Works

1. **Checkout**: The workflow checks out your module into a `module` directory and the d9-composer-managed upstream into an `upstream` directory
2. **Copy**: Your module is copied to `upstream/web/modules/packages/[module-name]`
3. **Setup**: DDEV is configured and Composer dependencies are installed with caching
4. **Validation**: composer.json and composer.lock are validated
5. **Install**: Drupal is installed using the ducore profile
6. **Enable**: Any specified modules are enabled via Drush
7. **Test**: PHPUnit tests are executed from the specified paths
8. **Notify**: On failure, an optional MS Teams notification is sent

## Related Resources

- [MS Teams Notification Action](../actions/ms-teams-notification/README.md)
- [PHPUnit D10 Reusable Workflow](./phpunit-d10-reusable-README.md)
- [Playwright D9 Reusable Workflow](./playwright-d9-reusable-README.md)
