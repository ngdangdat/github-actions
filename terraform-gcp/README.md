# Terraform GCP Module

A GitHub Action to run Terraform plan/apply with GCP Workload Identity authentication and tfcmt PR comments.

## Features

- Terraform plan and apply with PR comments via [tfcmt](https://github.com/suzuki-shunsuke/tfcmt)
- GCP Workload Identity for secure authentication (no static credentials)
- GCS backend support for Terraform state
- Flexible mode selection: `plan` or `apply`

## Usage

```yaml
# Plan only (default) - posts plan as PR comment
- uses: ngdangdat/github-actions/terraform-gcp@main
  with:
    gcp_workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    gcp_service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
    mode: plan

# Apply only - posts apply result as PR comment
- uses: ngdangdat/github-actions/terraform-gcp@main
  with:
    gcp_workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    gcp_service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
    mode: apply
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `gcp_workload_identity_provider` | GCP Workload Identity Provider | Yes | - |
| `gcp_service_account` | GCP Service Account email | Yes | - |
| `github_token` | GitHub token for tfcmt PR comments | Yes | - |
| `working_directory` | Directory containing Terraform configuration | No | `.` |
| `terraform_version` | Terraform version to use | No | `latest` |
| `mode` | Terraform mode: `plan` or `apply` | No | `plan` |
| `plan_args` | Additional arguments for terraform plan | No | - |
| `apply_args` | Additional arguments for terraform apply | No | `-auto-approve` |

## Example Workflow

```yaml
name: Terraform GCP

on:
  pull_request:
    paths:
      - 'terraform/**'
  push:
    branches:
      - main
    paths:
      - 'terraform/**'

permissions:
  contents: read
  id-token: write
  pull-requests: write  # Required for tfcmt to comment on PRs

jobs:
  plan:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: ngdangdat/github-actions/terraform-gcp@main
        with:
          gcp_workload_identity_provider: projects/123456/locations/global/workloadIdentityPools/my-pool/providers/github
          gcp_service_account: terraform@my-project.iam.gserviceaccount.com
          github_token: ${{ secrets.GITHUB_TOKEN }}
          working_directory: terraform
          terraform_version: "1.7.0"
          mode: plan

  apply:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: ngdangdat/github-actions/terraform-gcp@main
        with:
          gcp_workload_identity_provider: projects/123456/locations/global/workloadIdentityPools/my-pool/providers/github
          gcp_service_account: terraform@my-project.iam.gserviceaccount.com
          github_token: ${{ secrets.GITHUB_TOKEN }}
          working_directory: terraform
          terraform_version: "1.7.0"
          mode: apply
```

## Terraform Configuration

### Backend (GCS)

```hcl
terraform {
  backend "gcs" {
    bucket = "my-terraform-state-bucket"
    prefix = "gcp-infra"
  }
}
```

### Google Provider

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  # Uses Application Default Credentials from Workload Identity
  project = "my-project"
  region  = "us-central1"
}
```

## GCP Setup

1. Create a Workload Identity Pool and Provider:
   ```bash
   gcloud iam workload-identity-pools create "github-pool" \
     --location="global" \
     --display-name="GitHub Actions Pool"

   gcloud iam workload-identity-pools providers create-oidc "github-provider" \
     --location="global" \
     --workload-identity-pool="github-pool" \
     --display-name="GitHub Provider" \
     --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository" \
     --issuer-uri="https://token.actions.githubusercontent.com"
   ```

2. Grant the service account access:
   ```bash
   gcloud iam service-accounts add-iam-policy-binding "terraform@PROJECT.iam.gserviceaccount.com" \
     --role="roles/iam.workloadIdentityUser" \
     --member="principalSet://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/github-pool/attribute.repository/OWNER/REPO"
   ```

3. Ensure the service account has the necessary IAM roles for managing GCP resources and accessing the GCS state bucket.
