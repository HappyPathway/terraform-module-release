# Terraform Module Release Action
A GitHub Action that automates Terraform module validation, testing, formatting, and release management using semantic versioning based on PR titles.

## Features
- Runs `terraform init` to initialize the working directory
- Runs `terraform validate` to check configuration syntax
- Executes `terraform test` for module testing (skippable)
- Automatically formats code with `terraform fmt -recursive`
- Commits formatting changes if any are detected
- Creates semantic version tags based on PR title prefix
- Generates GitHub releases automatically
- Supports skipping tests via PR title or body flags

## Usage
To use this action, create a workflow that runs on pull request events. Your pull request title must start with one of these words:
- `Major`: For breaking changes (x.0.0)
- `Minor`: For new features (0.x.0)
- `Patch`: For bug fixes and minor changes (0.0.x)

### Skipping Tests
You can skip tests by including one of these flags in your PR title or body:
- `[skip ci]`
- `[skip test]`

This is particularly useful for formatting-only changes where running tests isn't necessary.

### Example Workflow

```yaml
name: Terraform CI/CD

on:
  pull_request:
    types: [closed]
    branches:
      - main

jobs:
  terraform-ci-cd:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    permissions:
      contents: write
    
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Run Terraform Module Release Action
        uses: your-org/terraform-module-release@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          working-directory: '.'

```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `github-token` | Yes | N/A | GitHub token for creating commits, tags, and releases |
| `working-directory` | No | `.` | Directory containing Terraform code |

## Behavior

1. When a PR is merged:
   - Initializes Terraform working directory
   - Validates Terraform configuration
   - Runs Terraform tests (unless skipped via flags)
   - Formats code and commits changes if needed (with [skip ci] [skip test] flags)
   - If PR title starts with Major/Minor/Patch:
     - Determines next version based on existing tags (starts from v0.0.0 if no tags exist)
     - Creates and pushes new tag
     - Creates GitHub release with auto-generated notes

2. If PR title doesn't start with Major/Minor/Patch:
   - Still performs initialization, validation, testing (unless skipped), and formatting
   - Skips version increment and release creation

3. If formatting changes are detected:
   - Changes are committed and pushed automatically
   - Version increment and release process is skipped for that run

## Requirements

- GitHub Actions runner with:
  - Git installed
  - Access to create commits, tags, and releases
  - Network access to download Terraform
  - GitHub CLI (gh) installed for PR and release operations