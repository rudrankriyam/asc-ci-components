# asc-ci-components

GitHub-released source preview GitLab CI/CD component templates for `asc`
(App Store Connect CLI).

Use this repository to install and run `asc` in iOS release pipelines, including App Store Connect and TestFlight automation.

## Publication status

These templates have a GitHub source release, but they are not published on
GitLab.com or in the GitLab CI/CD Catalog. Any component include that uses a
GitLab host is a future publication example, not an install path that works
today.

## Why this repository

- source templates for a future `asc` GitLab CI/CD integration
- reusable component templates built with `spec:inputs`
- GitHub source release tag `1.0.1`
- commit-SHA validation hooks for preview work
- self-test pipeline that validates components via `@$CI_COMMIT_SHA`

## Components

- `install`: installs a versioned `asc` binary and verifies checksums.
- `run`: runs a provided command with a checksum-verified `asc` binary. If a
  verified binary from an earlier `install` job is already present in the
  workspace (via artifacts), it is reused after re-verifying its checksum;
  otherwise the binary is downloaded and verified independently.

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
- `job_prefix` (default: `asc`, constrained to `^[a-zA-Z0-9][a-zA-Z0-9._-]*$`)
- `image` (default: `alpine:3.20`)
- `asc_version` (default: `latest`)
- `working_dir` (default: `$CI_PROJECT_DIR`)
- `profile` (optional)
- `bypass_keychain` (default: `"1"`)

### `run`

- `stage` (default: `deploy`)
- `job_prefix` (default: `asc`, constrained to `^[a-zA-Z0-9][a-zA-Z0-9._-]*$`)
- `image` (default: `alpine:3.20`)
- `asc_version` (default: `latest`)
- `command` (required)
- `working_dir` (default: `$CI_PROJECT_DIR`)
- `profile` (optional)
- `bypass_keychain` (default: `"1"`)

### Input handling notes

- `working_dir`, `asc_version`, `profile`, and `bypass_keychain` are passed to
  the job scripts through the job's `variables:` block and referenced as quoted
  shell variables, so their values are never spliced into shell syntax.
- `command` is captured via a quoted heredoc before execution.
- `job_prefix` is interpolated at the YAML level (it becomes part of the job
  name), so it is constrained by a `spec:inputs` regex to
  `^[a-zA-Z0-9][a-zA-Z0-9._-]*$`.
- The `run` component reuses a checksum-verified `asc` binary from a prior
  `install` job artifact when one is present in the workspace; otherwise it
  re-downloads and re-verifies independently.

## Pending publication examples

The examples below show the intended component syntax after a GitLab project is
created and catalog publication is done. Replace the host and namespace with
real published values before using them.

### 1) Single-job install + run

```yaml
include:
  - component: gitlab.example.com/your-group/asc-ci-components/run@1.0.1
    inputs:
      stage: deploy
      job_prefix: release
      asc_version: latest
      command: asc --help
```

Do not use `gitlab.com/rudrankriyam/asc-ci-components` unless that GitLab
project exists and contains the requested tag.

### 2) Staged pipeline usage

```yaml
stages: [prepare, deploy]

include:
  - component: gitlab.example.com/your-group/asc-ci-components/install@1.0.1
    inputs:
      stage: prepare
      job_prefix: asc-prepare
      asc_version: 0.31.5

  - component: gitlab.example.com/your-group/asc-ci-components/run@1.0.1
    inputs:
      stage: deploy
      job_prefix: asc-deploy
      asc_version: 0.31.5
      command: asc apps list --output json
```

### 3) Self-managed include form

```yaml
include:
  - component: "$CI_SERVER_FQDN/my-group/asc-ci-components/run@1.0.1"
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

## Publication checklist

1. Create or mirror this project on GitLab.
2. Validate the components in GitLab CI using commit-SHA includes.
3. Mirror or create the semantic version tag in the GitLab project, for example
   `1.0.1`.
4. Publish to the GitLab CI/CD Catalog if catalog discovery is wanted.

```bash
git tag 1.0.1
git push origin 1.0.1
```

The GitHub source release uses `1.0.1`; GitLab consumers still need a real
GitLab project and matching tag before the component include examples work.
