# Terraform GCP Reusable Workflow

Reusable workflow for Terraform plan/apply with GCP Workload Identity authentication and tfcmt PR comments.

## Usage

```yaml
name: Terraform GCP

on:
  pull_request:
    paths:
      - "terraform/gcp/**"
  push:
    branches:
      - main
    paths:
      - "terraform/gcp/**"

jobs:
  plan:
    if: github.event_name == 'pull_request'
    uses: ngdangdat/github-actions/.github/workflows/terraform-gcp.yml@main
    with:
      gcp_workload_identity_provider: projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/my-provider
      gcp_service_account: terraform@my-project.iam.gserviceaccount.com
      working_directory: terraform/gcp
      mode: plan

  apply:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    uses: ngdangdat/github-actions/.github/workflows/terraform-gcp.yml@main
    with:
      gcp_workload_identity_provider: projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/my-provider
      gcp_service_account: terraform@my-project.iam.gserviceaccount.com
      working_directory: terraform/gcp
      mode: apply
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

## Permissions

This workflow defines its own permissions:

- `contents: read` - Read repository contents
- `id-token: write` - Required for GCP Workload Identity
- `pull-requests: write` - Required for tfcmt PR comments

## Prerequisites

1. **GCP Workload Identity Federation** configured for GitHub Actions
2. **GCS backend** for Terraform state (configured in your Terraform code)

## Features

- GCP Workload Identity authentication (no service account keys)
- tfcmt for rich PR comments with plan output
- aqua for tool management with caching
- Supports custom Terraform version
