# How to Deploy to Azure Static Web Apps

This guide walks you through deploying the Invoice Processing app to Azure Static Web Apps with an Azure Functions API backed by Cosmos DB.

## Architecture

```
Azure Static Web App (Standard tier)
├── Frontend: React + Vite (dist/)
└── Linked Backend: Azure Function App
    └── Cosmos DB for NoSQL (Entra auth, no local keys)
```

All authentication between services uses **managed identities** (Entra ID). No connection strings or shared keys are stored.

## Prerequisites

| Requirement | Details |
|---|---|
| **Node.js** | Version 20.0 or later |
| **Azure CLI** | Version 2.60 or later ([Install guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)) |
| **Azure Functions Core Tools** | Version 4 (`npm install -g azure-functions-core-tools@4`) |
| **SWA CLI** | Version 2.x (`npm install -g @azure/static-web-apps-cli`) |
| **Azure subscription** | [Create a free account](https://azure.microsoft.com/pricing/purchase-options/azure-account) if you don't have one |

## 1. Install Dependencies

```bash
npm install
cd api && npm install && cd ..
```

## 2. Sign In to Azure

```bash
az login
```

If you have multiple subscriptions, select the correct one:

```bash
az account set --subscription "7c71b563-0dc0-4bc0-bcf6-06f8f0516c7a"
```

## 3. Create Azure Resources (First-Time Only)

> **NOTE:** These resources already exist for the current deployment. Only follow this section if you want another instance.

### Resource Group

```bash
az group create -n invoice-processing -l centralus
```

### Static Web App (Standard tier)

```bash
az staticwebapp create -n invoice-processing-s -g invoice-processing --sku Standard
```

Enable its system-assigned managed identity:

```bash
az staticwebapp identity assign -n invoice-processing-s -g invoice-processing
```

### Cosmos DB (Entra auth only)

```bash
az cosmosdb create \
  --name invoice-processing-cosmos \
  --resource-group invoice-processing \
  --locations regionName=centralus \
  --capabilities EnableServerless \
  --disable-local-auth true

az cosmosdb sql database create \
  --account-name invoice-processing-cosmos \
  --resource-group invoice-processing \
  --name InvoiceDB

az cosmosdb sql container create \
  --account-name invoice-processing-cosmos \
  --resource-group invoice-processing \
  --database-name InvoiceDB \
  --name Invoices \
  --partition-key-path /id
```

### Function App (Flex Consumption)

Create a storage account (shared key access disabled per policy):

```bash
az storage account create \
  --name invoiceprocfuncstor \
  --resource-group invoice-processing \
  --location centralus \
  --sku Standard_LRS \
  --allow-shared-key-access false
```

Create the Function App:

```bash
az functionapp create \
  --name invoice-processing-func \
  --resource-group invoice-processing \
  --flexconsumption-location centralus \
  --runtime node --runtime-version 20 \
  --functions-version 4 \
  --storage-account invoiceprocfuncstor
```

Enable managed identity:

```bash
az functionapp identity assign \
  --name invoice-processing-func \
  --resource-group invoice-processing
```

### RBAC Assignments

Grant the Function App's managed identity access to Cosmos DB (Built-in Data Contributor):

```bash
FUNC_PRINCIPAL=$(az functionapp identity show \
  --name invoice-processing-func \
  --resource-group invoice-processing \
  --query principalId -o tsv)

az cosmosdb sql role assignment create \
  --account-name invoice-processing-cosmos \
  --resource-group invoice-processing \
  --scope "/" \
  --principal-id $FUNC_PRINCIPAL \
  --role-definition-id "00000000-0000-0000-0000-000000000002"
```

Grant storage roles for the Function App runtime:

```bash
STORAGE_ID=$(az storage account show \
  --name invoiceprocfuncstor \
  --resource-group invoice-processing \
  --query id -o tsv)

az role assignment create --assignee $FUNC_PRINCIPAL --role "Storage Blob Data Owner" --scope $STORAGE_ID
az role assignment create --assignee $FUNC_PRINCIPAL --role "Storage Queue Data Contributor" --scope $STORAGE_ID
az role assignment create --assignee $FUNC_PRINCIPAL --role "Storage Table Data Contributor" --scope $STORAGE_ID
```

Configure identity-based deployment storage (via REST API since Azure Policy blocks key-based auth):

```bash
TOKEN=$(az account get-access-token --query accessToken -o tsv)
FUNC_RESOURCE_ID="/subscriptions/7c71b563-0dc0-4bc0-bcf6-06f8f0516c7a/resourceGroups/invoice-processing/providers/Microsoft.Web/sites/invoice-processing-func"

curl -s -X PATCH \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"properties":{"functionAppConfig":{"deployment":{"storage":{"type":"blobContainer","value":"https://invoiceprocfuncstor.blob.core.windows.net/app-package-invoiceprocessingfunc-9303679","authentication":{"type":"SystemAssignedIdentity"}}},"runtime":{"name":"node","version":"20"},"scaleAndConcurrency":{"maximumInstanceCount":100,"instanceMemoryMB":2048}}}}' \
  "https://management.azure.com${FUNC_RESOURCE_ID}?api-version=2024-04-01"
```

### Link Function App to SWA

```bash
TOKEN=$(az account get-access-token --query accessToken -o tsv)
SWA_RESOURCE="/subscriptions/7c71b563-0dc0-4bc0-bcf6-06f8f0516c7a/resourceGroups/invoice-processing/providers/Microsoft.Web/staticSites/invoice-processing-s"
FUNC_RESOURCE="/subscriptions/7c71b563-0dc0-4bc0-bcf6-06f8f0516c7a/resourceGroups/invoice-processing/providers/Microsoft.Web/sites/invoice-processing-func"

curl -s -X PUT \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"properties\":{\"backendResourceId\":\"${FUNC_RESOURCE}\",\"region\":\"centralus\"}}" \
  "https://management.azure.com${SWA_RESOURCE}/linkedBackends/backend1?api-version=2023-12-01"
```

## 4. Build the Frontend

```bash
npm run swa:build
```

This runs the Vite build and outputs the distributable files to the `dist/` directory.

## 5. Authenticate with SWA CLI

```bash
npm run swa:login
```

This logs you into Azure through the SWA CLI and saves credentials locally in a `.env` file (git-ignored).

## 6. Deploy the Frontend

```bash
npm run swa:deploy
```

When prompted, select `invoice-processing/invoice-processing-s` from the list.

Once complete, the deployed URL will be displayed:

```
✔ Project deployed to https://thankful-moss-03f406b10.6.azurestaticapps.net 🚀
```

## 7. Deploy the API (Azure Functions)

```bash
cd api
func azure functionapp publish invoice-processing-func --javascript
cd ..
```

This deploys the functions to the linked Function App. The API will be accessible at `/api/*` through the SWA URL.

## Quick Reference

```bash
# Frontend deploy
npm run swa:build
npm run swa:login
npm run swa:deploy

# API deploy
cd api && func azure functionapp publish invoice-processing-func --javascript && cd ..
```

## Project Structure

| Path | Purpose |
|---|---|
| `src/` | React frontend source code |
| `api/` | Azure Functions API (v3 programming model, Node.js) |
| `api/invoices/` | GET + POST `/api/invoices` |
| `api/invoices-update/` | PUT `/api/invoices/{id}` |
| `api/shared/cosmos.js` | Cosmos DB client (uses `DefaultAzureCredential`) |
| `staticwebapp.config.json` | SPA fallback and route exclusions |
| `swa-cli.config.json` | SWA CLI build and deployment configuration |

## Azure Resources

| Resource | Name | Purpose |
|---|---|---|
| Resource Group | `invoice-processing` | Contains all resources |
| Static Web App | `invoice-processing-s` | Hosts the React frontend |
| Function App | `invoice-processing-func` | API backend (Flex Consumption, Node.js 20) |
| Cosmos DB | `invoice-processing-cosmos` | Invoice data store (serverless, Entra-only) |
| Storage Account | `invoiceprocfuncstor` | Function App runtime storage (identity-based) |

## Troubleshooting

- **Not logged in**: Run `az login` first, then `npm run swa:login`.
- **Wrong subscription**: Use `az account show` to verify, then `az account set` to switch.
- **Build errors**: Ensure Node.js 20+ is installed and run `npm install` before building.
- **API returns 404**: Ensure the Function App is linked as a backend to the SWA and functions are deployed.
- **API returns 500**: Check Function App logs with `func azure functionapp logstream invoice-processing-func`. Verify the managed identity has Cosmos DB Data Contributor role.
- **Function deploy fails with storage error**: The storage account requires identity-based auth. Ensure the deployment storage is configured with `SystemAssignedIdentity` (see step 3).
