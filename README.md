# GitHub Actions Workflow for Listing GCP VMs

## Overview
This GitHub Actions workflow allows users to manually trigger a workflow using `workflow_dispatch`, providing inputs such as `project_id`, `stack_name`, `stack_environment`, and `service_name`. The workflow authenticates to Google Cloud using a service account and lists VM instances based on the provided filters.

## Workflow Definition

```yaml
name: List GCP VMs

on:
  workflow_dispatch:
    inputs:
      project_id:
        description: 'GCP Project ID'
        required: true
        default: 'rishabhmodi'
      stack_name:
        description: 'Stack Name'
        required: true
      stack_environment:
        description: 'Stack Environment'
        required: true
      service_name:
        description: 'Service Name'
        required: true

jobs:
  list-vms:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
      
      - name: Authenticate to GCP
        uses: google-github-actions/auth@v2
        with:
          credentials_json: '${{ secrets.GCP_SA_KEY }}'
      
      - name: Set up gcloud SDK
        uses: google-github-actions/setup-gcloud@v2
      
      - name: List VMs in GCP
        run: |
          gcloud config set project ${{ github.event.inputs.project_id }}
          gcloud compute instances list --filter="name:${{ github.event.inputs.stack_name }}-${{ github.event.inputs.stack_environment }}-${{ github.event.inputs.service_name }}-*" --format="table(name, zone, status)"
```

## Setup Instructions
### 1. Create a Google Service Account
- Navigate to the [Google Cloud IAM & Admin Console](https://console.cloud.google.com/iam-admin/serviceaccounts).
- Select your project (`tms`).
- Click **Create Service Account** and provide a suitable name.
- Assign the following roles:
  - `Compute Viewer` (`roles/compute.viewer`) – To list VM instances.
  - `Service Account Token Creator` (`roles/iam.serviceAccountTokenCreator`) – Required for authentication.
- Click **Create & Continue**.

### 2. Generate and Configure Service Account Key
- Open the created service account.
- Navigate to the **Keys** tab and click **Add Key** > **Create New Key**.
- Select **JSON** format and click **Create**.
- The key file will be downloaded to your machine.
- Go to **GitHub Repository Settings** > **Secrets and Variables** > **Actions**.
- Add a new secret with the name **GCP_SA_KEY** and paste the JSON key content.

### 3. Running the Workflow
- Navigate to the **GitHub Actions** tab.
- Select the **List GCP VMs** workflow.
- Click **Run Workflow**.
- Enter the required parameters and execute.

### 4. Verifying Results
- The workflow will list matching VM instances in the job logs.
- To verify manually, run the following command:
  ```sh
  gcloud compute instances list --filter="name:<stack_name>-<stack_environment>-<service_name>-*"
  ```

This document provides a step-by-step guide to setting up and using the GitHub Actions workflow for listing GCP VMs. For any issues, refer to GitHub Actions logs or GCP console logs.
