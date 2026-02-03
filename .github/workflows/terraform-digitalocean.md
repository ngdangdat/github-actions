# Terraform DigitalOcean Reusable Workflow

Reusable workflow for Terraform plan/apply with DigitalOcean provider authentication, GCS backend via Workload Identity, and tfcmt PR comments.

## Usage

```yaml
name: Terraform DigitalOcean

on:
  pull_request:
    paths:
      - "terraform/digitalocean/**"
  push:
    branches:
      - main
    paths:
      - "terraform/digitalocean/**"

jobs:
  plan:
    if: github.event_name == 'pull_request'
    uses: ngdangdat/github-actions/.github/workflows/terraform-digitalocean.yml@main
    with:
      gcp_workload_identity_provider: projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/github
      gcp_service_account: terraform@my-project.iam.gserviceaccount.com
      working_directory: terraform/digitalocean
      mode: plan
    secrets:
      digitalocean_token: ${{ secrets.DIGITALOCEAN_TOKEN }}
      spaces_access_key: ${{ secrets.SPACES_ACCESS_KEY }}
      spaces_secret_key: ${{ secrets.SPACES_SECRET_KEY }}

  apply:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    uses: ngdangdat/github-actions/.github/workflows/terraform-digitalocean.yml@main
    with:
      gcp_workload_identity_provider: projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/github
      gcp_service_account: terraform@my-project.iam.gserviceaccount.com
      working_directory: terraform/digitalocean
      mode: apply
    secrets:
      digitalocean_token: ${{ secrets.DIGITALOCEAN_TOKEN }}
      spaces_access_key: ${{ secrets.SPACES_ACCESS_KEY }}
      spaces_secret_key: ${{ secrets.SPACES_SECRET_KEY }}
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `gcp_workload_identity_provider` | GCP Workload Identity Provider for GCS backend | Yes | - |
| `gcp_service_account` | GCP Service Account email for Workload Identity | Yes | - |
| `working_directory` | Directory containing Terraform configuration | No | `.` |
| `terraform_version` | Terraform version to use | No | `latest` |
| `mode` | Terraform mode: `plan` or `apply` | No | `plan` |
| `plan_args` | Additional arguments for terraform plan | No | `""` |
| `apply_args` | Additional arguments for terraform apply | No | `-auto-approve` |

## Secrets

| Name | Description | Required |
|------|-------------|----------|
| `digitalocean_token` | DigitalOcean API Token for Terraform provider | Yes |
| `spaces_access_key` | DigitalOcean Spaces access key (for future use) | No |
| `spaces_secret_key` | DigitalOcean Spaces secret key (for future use) | No |

## Permissions

This workflow defines its own permissions:

- `contents: read` - Read repository contents
- `id-token: write` - Required for GCP Workload Identity authentication
- `pull-requests: write` - Required for tfcmt PR comments

## Prerequisites

1. **GCP Workload Identity** configured for GitHub Actions (for GCS backend)
2. **GCS Bucket** for Terraform state with appropriate IAM permissions
3. **DigitalOcean API Token** with appropriate permissions for your resources

## Terraform Backend Configuration

To use GCS as your Terraform backend:

```hcl
terraform {
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "digitalocean"
  }
}
```

## Features

- DigitalOcean API token authentication
- GCS backend support via GCP Workload Identity
- tfcmt for rich PR comments with plan output
- aqua for tool management with caching
- Supports custom Terraform version
