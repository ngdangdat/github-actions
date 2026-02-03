# Terraform DigitalOcean Reusable Workflow

Reusable workflow for Terraform plan/apply with DigitalOcean provider authentication and tfcmt PR comments.

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
| `working_directory` | Directory containing Terraform configuration | No | `.` |
| `terraform_version` | Terraform version to use | No | `latest` |
| `mode` | Terraform mode: `plan` or `apply` | No | `plan` |
| `plan_args` | Additional arguments for terraform plan | No | `""` |
| `apply_args` | Additional arguments for terraform apply | No | `-auto-approve` |

## Secrets

| Name | Description | Required |
|------|-------------|----------|
| `digitalocean_token` | DigitalOcean API Token for Terraform provider | Yes |
| `spaces_access_key` | DigitalOcean Spaces access key for S3 backend | No |
| `spaces_secret_key` | DigitalOcean Spaces secret key for S3 backend | No |

## Permissions

This workflow defines its own permissions:

- `contents: read` - Read repository contents
- `pull-requests: write` - Required for tfcmt PR comments

## Prerequisites

1. **DigitalOcean API Token** with appropriate permissions for your resources
2. **DigitalOcean Spaces** (optional) for Terraform state backend with access keys

## Terraform Backend Configuration

To use DigitalOcean Spaces as your Terraform backend:

```hcl
terraform {
  backend "s3" {
    endpoints = {
      s3 = "https://nyc3.digitaloceanspaces.com"
    }
    bucket                      = "my-terraform-state"
    key                         = "terraform.tfstate"
    region                      = "us-east-1"  # Required but ignored by DO Spaces
    skip_credentials_validation = true
    skip_requesting_account_id  = true
    skip_metadata_api_check     = true
    skip_s3_checksum            = true
  }
}
```

## Features

- DigitalOcean API token authentication
- Optional Spaces backend support (S3-compatible)
- tfcmt for rich PR comments with plan output
- aqua for tool management with caching
- Supports custom Terraform version
