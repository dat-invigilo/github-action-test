# GitHub Action Test: Sync Configs to Azure Blob Storage

## Overview
This repository manages configuration files stored in the `test artifacts/` directory. It features a GitHub Actions workflow that automatically synchronizes any modified `.yaml` or `.yml` files to an Azure Blob Storage container (specifically into the `prod-deployment-artifacts` folder) whenever changes are merged into the `main` branch.

## Setup & Authentication

To allow GitHub Actions to upload files to your Azure Blob Storage securely, you need to create a Service Principal (a bot user) in Azure with the necessary permissions.

### 1. Create the Azure Service Principal

Run the following Azure CLI command in your terminal to create a Service Principal with the **Storage Blob Data Contributor** role, granting it access to your specific resource group:

```bash
az ad sp create-for-rbac \
  --name "GitHub-Action-Storage-Sync" \
  --role "Storage Blob Data Contributor" \
  --scopes /subscriptions/<YOUR_SUBSCRIPTION_ID>/resourceGroups/invigilo \
  --sdk-auth
```

*Note: The `--sdk-auth` flag causes the command to output a JSON object containing the credentials.*

### 2. Configure GitHub Secrets

Copy the **entire JSON output** from the previous step and go to your GitHub repository -> **Settings** -> **Secrets and variables** -> **Actions**. 

Add the following Repository Secrets:

*   `AZURE_CREDENTIALS`: Paste the exact JSON output from the `az ad sp` command. It will be in this format:
    ```json
    {
      "clientId": "your AppID",
      "clientSecret": "your_actual_password",
      "tenantId": "your_actual_tenant_id"
    }
    ```
*   `AZURE_STORAGE_ACCOUNT_NAME`: The name of your target Azure Storage Account.
*   `AZURE_STORAGE_CONTAINER_NAME`: The name of your target Azure Blob container.

## How It Works

1. You edit or add `.yaml` files inside the `test artifacts/` folder.
2. A Pull Request containing these changes is merged into the `main` branch.
3. The GitHub Actions workflow (`.github/workflows/sync-to-azure-blob.yml`) is triggered.
4. The workflow authenticates to Azure using the `AZURE_CREDENTIALS` secret.
5. It runs the `az storage blob upload-batch` command to sync only the `.yaml` files over to the `prod-deployment-artifacts` path in your targeted Blob container, overwriting the old files.