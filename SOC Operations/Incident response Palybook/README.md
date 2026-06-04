# Malware EDR Alerts Response Playbook

This repository contains an ARM template for a Microsoft Sentinel playbook that responds to malware alerts from Defender for Endpoint.

The playbook processes incident alerts and can:

1. Query Log Analytics for host and threat details.
2. Isolate impacted endpoint devices.
3. Collect and download investigation packages.
4. Upload investigation package files to Azure Blob Storage.
5. Add incident comments back into Microsoft Sentinel.

## Template File

Use this template for deployment:

1. Malware-EDR-Alerts-Response.json

## Parameters

| Parameter | Required | Default | Description |
|---|---|---|---|
| PlaybookName | No | Malware-EDR-Alerts-Response | Logic App playbook name |
| AzureSentinelConnectionName | No | azuresentinel | Microsoft Sentinel API connection name |
| AzureMonitorLogsConnectionName | No | azuremonitorlogs | Azure Monitor Logs API connection name |
| AzureBlobConnectionName | No | azureblob | Azure Blob API connection name |
| StorageAccountName | Yes | n/a | Storage account used to store investigation packages |

## What The Template Deploys

1. Azure Monitor Logs API connection (Microsoft.Web/connections)
2. Azure Blob API connection (Microsoft.Web/connections)
3. Logic App playbook with system-assigned identity (Microsoft.Logic/workflows)

Note: The Sentinel connection is referenced by name and must already exist in the target resource group.

## Prerequisites

1. Microsoft Sentinel enabled on the target workspace.
2. Defender for Endpoint integrated with Sentinel alerts.
3. Existing Sentinel API connection with the name in AzureSentinelConnectionName.
4. Storage account already created for package upload.
5. Permissions to deploy Logic Apps and API connections in the target resource group.

## Deploy

### Azure Portal

1. Open Custom deployment in the target resource group.
2. Upload Malware-EDR-Alerts-Response.json.
3. Fill in parameters, especially StorageAccountName.
4. Run deployment.

### Azure CLI

```bash
az deployment group create \
  --resource-group <RESOURCE-GROUP> \
  --template-file Malware-EDR-Alerts-Response.json \
  --parameters StorageAccountName=<STORAGE-ACCOUNT-NAME>
```

## Post-Deployment Steps

### 1. Authorize API Connections

Authorize all used connections in the resource group:

1. azuresentinel
2. azuremonitorlogs
3. azureblob

For each connection:

1. Open API connection resource.
2. Select Edit API connection.
3. Select Authorize.
4. Save.

### 2. Grant Managed Identity Permissions

The Logic App uses managed identity for Defender for Endpoint API calls and must have required permissions in your tenant.

Typical permissions/scopes required for this playbook flow include:

1. Defender for Endpoint machine read and response action permissions.
2. Access to perform machine isolation and collect investigation package actions.

Also grant storage write access if needed based on your connector authentication model.

### 3. Validate Triggering

1. Ensure Sentinel automation rule triggers this playbook on malware incidents.
2. Confirm incident alerts include expected host entities.

## Notes

1. The playbook loops through incident alerts and applies containment per alert.
2. Investigation packages are saved under folder prefix mde-investigation-packages/<HostName>.
3. If redeploying from deployment history, old parameter values may be reused.

## Validate Before Deploying

```bash
az deployment group validate \
  --resource-group <RESOURCE-GROUP> \
  --template-file Malware-EDR-Alerts-Response.json \
  --parameters StorageAccountName=<STORAGE-ACCOUNT-NAME>
```
