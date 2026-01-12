# Terraform GitHub Reusable Workflow

Reusable workflow for Terraform plan/apply with GitHub App authentication (for Terraform GitHub provider), GCS backend via Workload Identity, and tfcmt PR comments.

## Usage

```yaml
name: Terraform GitHub

on:
  pull_request:
    paths:
      - "terraform/github/**"
  push:
    branches:
      - main
    paths:
      - "terraform/github/**"

jobs:
  plan:
    if: github.event_name == 'pull_request'
    uses: ngdangdat/github-actions/.github/workflows/terraform-github.yml@main
    with:
      gcp_workload_identity_provider: projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/my-provider
      gcp_service_account: terraform@my-project.iam.gserviceaccount.com
      working_directory: terraform/github
      mode: plan
    secrets:
      gh_app_id: ${{ secrets.GH_APP_ID }}
      gh_app_private_key: ${{ secrets.GH_APP_PRIVATE_KEY }}
      github_app_installation_id: ${{ secrets.GH_APP_INSTALLATION_ID }}

  apply:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    uses: ngdangdat/github-actions/.github/workflows/terraform-github.yml@main
    with:
      gcp_workload_identity_provider: projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/my-provider
      gcp_service_account: terraform@my-project.iam.gserviceaccount.com
      working_directory: terraform/github
      mode: apply
    secrets:
      gh_app_id: ${{ secrets.GH_APP_ID }}
      gh_app_private_key: ${{ secrets.GH_APP_PRIVATE_KEY }}
      github_app_installation_id: ${{ secrets.GH_APP_INSTALLATION_ID }}
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `gcp_workload_identity_provider` | GCP Workload Identity Provider | Yes | - |
| `gcp_service_account` | GCP Service Account email for Workload Identity | Yes | - |
| `working_directory` | Directory containing Terraform configuration | No | `.` |
| `terraform_version` | Terraform version to use | No | `latest` |
| `mode` | Terraform mode: `plan` or `apply` | No | `plan` |
| `plan_args` | Additional arguments for terraform plan | No | `""` |
| `apply_args` | Additional arguments for terraform apply | No | `-auto-approve` |

## Secrets

| Name | Description | Required |
|------|-------------|----------|
| `gh_app_id` | GitHub App ID for Terraform GitHub provider | Yes |
| `gh_app_private_key` | GitHub App Private Key (PEM format) | Yes |
| `github_app_installation_id` | GitHub App Installation ID | Yes |

## Permissions

This workflow defines its own permissions:

- `contents: read` - Read repository contents
- `id-token: write` - Required for GCP Workload Identity
- `pull-requests: write` - Required for tfcmt PR comments

## Prerequisites

1. **GCP Workload Identity Federation** configured for GitHub Actions
2. **GCS backend** for Terraform state (configured in your Terraform code)
3. **GitHub App** with permissions to manage the resources you're terraforming

## GitHub App Setup

The GitHub App needs these permissions for the Terraform GitHub provider:

- Repository permissions as needed (admin, contents, etc.)
- Organization permissions as needed (members, teams, etc.)

The app credentials are passed as environment variables:
- `GITHUB_APP_ID`
- `GITHUB_APP_INSTALLATION_ID`
- `GITHUB_APP_PEM_FILE`

## Features

- GitHub App authentication for Terraform GitHub provider
- GCP Workload Identity authentication for GCS backend
- tfcmt for rich PR comments with plan output
- aqua for tool management with caching
- Supports custom Terraform version
