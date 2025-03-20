# tf-buildtemplate

Reusable template for GitHub Workflows that manage Terraform-based Infrastructure-as-Code (IaC).

## Overview

The **tf-buildtemplate** repository provides a set of workflow templates that initialize Terraform with an Azure backend, run Terraform plan, and execute the apply step. The goal is enable standardization of how Terraform is used across an organization, ensuring consistency, reducing operational overhead, and minimizing divergence in IaC practices.

By centralizing and reusing these workflow templates, teams can:
- **Ensure Consistency:** All Terraform-based deployments follow a unified, best-practice approach.
- **Simplify Maintenance:** Updates or improvements to the Terraform workflow can be centralized, reducing the need for each team to manage separate implementations.
- **Promote Standardization:** Repositories configured in a uniform manner will automatically benefit from shared, proven setups for Terraform initialization, planning, and application.

## How to Use the Template

### v1

#### Prerequisites

- **Repository Directory Structure:**  
  Repositories using this template should follow a consistent layout. For example, place your Terraform configuration files in a dedicated directory (e.g., `./TF`).

- **Azure Backend Setup:**  
  Ensure that you have an Azure Storage Account and a Resource Group prepared to host your Terraform state. These values will be referenced in your workflow.
  Make sure you have a Service Principal with permissions to deploy resources to your azure subscription, and Storage Blob Data Contributor Role on the Storage Account.
  Set up (federated credentials)[https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure-openid-connect] for your service principal.

- **GitHub Variables and Secrets:**  
  Set up required environment variables or repository secrets such as:
  - `CLIENTID` (Azure Service Principal Client ID)
  - `TENANTID` (Azure Active Directory Tenant ID)
  - `SUBSCRIPTIONID` (Azure Subscription ID)
  - Additional variables for your backend configuration (e.g., storage account names, resource group names)


#### Example Workflow Usage

Create a workflow file in your repository (e.g., `.github/workflows/deploy-functionapp.yml`) that calls the reusable workflows provided by the template:

```yaml
name: Deploy FunctionApp via Template

permissions:
  id-token: write
  contents: read

on:
  pull_request:
    types: [opened, synchronize, reopened]
    paths:
      - "TF/FunctionApp/**"
  push:
    branches: [main]
    paths:
      - "TF/FunctionApp/**"
  workflow_dispatch:

jobs:
  azure-login:
    # Adjust the runner settings as needed.
    runs-on: ubuntu-latest
    # Optionally specify your environment.
    environment: your-environment-name
    steps:
      - name: Azure Login (Federated credentials)
        uses: azure/login@v2
        with:
          client-id: ${{ vars.CLIENTID }}  # Azure Service Principal Client ID
          tenant-id: ${{ vars.TENANTID }}   # Azure Active Directory Tenant ID
          enable-AzPSSession: true
          allow-no-subscriptions: true

  terraform:
    needs: azure-login
    uses: your-org/tf-buildtemplate/.github/workflows/tf-plan-apply.yml@v1.0.0
    with:
      tfDirectory: "./TF/FunctionApp"   # Directory containing the Terraform configuration
      jobEnvironment: "your-job-environment"  # e.g., "Production-plan" or "Test-plan"
      tfStateKey: "function.tfstate"     # Name for the Terraform state file