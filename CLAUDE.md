# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

A collection of reusable GitHub Actions (composite actions) for infrastructure automation workflows. Each action is self-contained in its own directory with an `action.yml` and `README.md`.

## Actions

- **install-gh/**: Install GitHub CLI across Linux, macOS, Windows runners using aqua
- **terraform-github/**: Terraform plan/apply with GitHub App auth, GCS backend via Workload Identity, tfcmt PR comments
- **terraform-gcp/**: Terraform plan/apply with GCP Workload Identity auth, GCS backend, tfcmt PR comments

## Claude Commands

### /gh:create-release-tag

Create and push a release tag for workflows to reference.

Instructions:
1. Ask the user for the version number if not provided as argument
2. Validate the version follows semver format (e.g., 1.0.0, 1.2.3)
3. Create the git tag with `v` prefix (e.g., v1.0.0)
4. Push the tag to origin

Usage in workflows: `uses: ngdangdat/github-actions/<action>@v<version>`

## Action Development

Each action follows this structure:
```
<action-name>/
├── action.yml    # Composite action definition (using: composite)
├── README.md     # Usage docs with inputs table and examples
```

Key dependencies used across actions:
- **aqua**: Fast tool installation with caching
- **tfcmt**: Terraform plan/apply PR comments
- **google-github-actions/auth**: GCP Workload Identity auth
- **hashicorp/setup-terraform**: Terraform version management

## Commit Convention

Format: `<type>: <description>` (single line)

Types: `feat`, `chore`, `doc`, `fix`, `infra`
