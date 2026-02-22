# asc-ci-components

Official GitLab CI/CD components for `asc` (App Store Connect CLI).

Catalog-ready GitLab CI/CD component templates for GitLab.com and self-managed GitLab.
Use this repository to install and run `asc` in iOS release pipelines, including App Store Connect and TestFlight automation.

## Why this repository

- official integration surface for `asc` on GitLab CI/CD
- reusable component templates built with `spec:inputs`
- versioned includes (`@1.0.0`) for stable production pipelines
- self-test pipeline that validates components via `@$CI_COMMIT_SHA`

## Components

- `install`: installs a versioned `asc` binary and verifies checksums.
- `run`: installs a versioned `asc` binary and executes a provided command.

## Repository Layout

```text
templates/
  install.yml
  run.yml
.gitlab-ci.yml
README.md
LICENSE.md
```

## Inputs

### `install`

- `stage` (default: `deploy`)
- `job_prefix` (default: `asc`)
- `asc_version` (default: `latest`)
- `working_dir` (default: `$CI_PROJECT_DIR`)
- `profile` (optional)
- `bypass_keychain` (default: `"1"`)

### `run`

- `stage` (default: `deploy`)
- `job_prefix` (default: `asc`)
- `asc_version` (default: `latest`)
- `command` (required)
- `working_dir` (default: `$CI_PROJECT_DIR`)
- `profile` (optional)
- `bypass_keychain` (default: `"1"`)

## Usage Examples

### 1) Single-job install + run

```yaml
include:
  - component: gitlab.com/rudrankriyam/asc-ci-components/run@1.0.0
    inputs:
      stage: deploy
      job_prefix: release
      asc_version: latest
      command: asc --help
```

For production pipelines, pin to semver tags. Use `@main` only for preview/testing.

### 2) Staged pipeline usage

```yaml
stages: [prepare, deploy]

include:
  - component: gitlab.com/rudrankriyam/asc-ci-components/install@1.0.0
    inputs:
      stage: prepare
      job_prefix: asc-prepare
      asc_version: 0.31.5

  - component: gitlab.com/rudrankriyam/asc-ci-components/run@1.0.0
    inputs:
      stage: deploy
      job_prefix: asc-deploy
      asc_version: 0.31.5
      command: asc apps list --output json
```

### 3) Self-managed include form

```yaml
include:
  - component: "$CI_SERVER_FQDN/my-group/asc-ci-components/run@1.0.0"
    inputs:
      stage: deploy
      job_prefix: internal-release
      asc_version: latest
      command: asc --help
```

## Component Validation

This project includes its own components in `.gitlab-ci.yml` using:

- `component: "$CI_SERVER_FQDN/$CI_PROJECT_PATH/<component>@$CI_COMMIT_SHA"`

The validation flow also calls the GitLab API through `$CI_API_V4_URL` to confirm included jobs are present and executable.

This follows GitLab component guidance around parameterized templates and commit-SHA validation in the component project itself.

## Release Process

1. Merge changes to `main`.
2. Tag a semantic version (for example `1.0.0`).
3. Push the tag and publish release notes.

```bash
git tag 1.0.0
git push origin 1.0.0
```

Use semantic version tags for catalog consumption and stable downstream includes.
