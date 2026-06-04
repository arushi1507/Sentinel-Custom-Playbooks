# Watchlist Upload Playbook ARM Template

This repository includes an ARM template for a Microsoft Sentinel watchlist upload playbook.
The playbook runs on a schedule, reads CSV files from a SharePoint folder, and upserts each file into Sentinel as a watchlist.

## Template File

Use this template:

1. Watchlist Deployment.json

## Playbook Flow

1. Trigger on recurrence (every 12 months by default).
2. List files from the configured SharePoint folder.
3. Read each file content.
4. Build a sanitized watchlist alias from file name.
5. Push CSV content to Sentinel Watchlists REST API using managed identity.

## Parameters

| Parameter | Required | Default | Description |
|---|---|---|---|
| workflows_WatchlistUpload_NonProd_name | No | WatchlistUpload_NonProd | Logic App workflow name |
| location | No | [resourceGroup().location] | Deployment region |
| subscriptionID | No | [subscription().subscriptionId] | Target subscription for Sentinel workspace |
| resourceGroupName | No | [resourceGroup().name] | Resource group containing Sentinel workspace |
| workspaceName | Yes | "" | Log Analytics workspace name used by Sentinel |
| sharepointConnectionName | No | sharepointonline-1 | SharePoint API connection resource name |
| sharePointSiteUrl | Yes | Project default URL | SharePoint site URL containing watchlist files |
| sharePointWatchlistFolder | Yes | Project default folder | SharePoint folder path containing CSV files |
| connections_sharepointonline_1_externalid | No | [resourceId('Microsoft.Web/connections', parameters('sharepointConnectionName'))] | SharePoint connection resource ID |

## What The Template Deploys

1. Microsoft.Web/connections for SharePoint Online.
2. Microsoft.Logic/workflows with SystemAssigned managed identity.

The workflow has dependsOn for the SharePoint API connection so the connection resource is created first.

## Prerequisites

1. Microsoft Sentinel enabled in the target workspace.
2. Permission to deploy Logic Apps and API connections in the target resource group.
3. SharePoint site and folder with CSV watchlist files.

## Permissions Required

### Deployment Identity

1. Contributor (or equivalent) on target resource group.
2. Permission to create Microsoft.Web/connections.
3. Sentinel Contributor on the target Microsoft Sentinel workspace.

### API Connection Authorization Account

1. Access to the target SharePoint site and folder.
2. Permission to authorize SharePoint connector consent in Azure.

### Logic App Managed Identity (Runtime)

1. Microsoft Sentinel watchlists write/update permission on target workspace.

## Deploy

### Azure Portal

1. Open Custom deployment in the target resource group.
2. Upload Watchlist Deployment.json.
3. Provide parameter values for workspaceName, sharePointSiteUrl, and sharePointWatchlistFolder.
4. Deploy.

### Azure CLI

```bash
az deployment group create \
  --resource-group <RESOURCE-GROUP> \
  --template-file "Watchlist Deployment.json"
```

## Post-Deployment Steps

### 1. Authorize SharePoint Connection

1. Open Resource Group -> API connections.
2. Open the deployed SharePoint connection (default: sharepointonline-1).
3. Select Edit API connection -> Authorize -> Save.

### 2. Enable and Test

1. Confirm Logic App is enabled.
2. Run trigger once manually or wait for schedule.
3. Verify watchlists were created or updated in Sentinel.

## Notes

1. If the connection is not authorized, workflow API actions fail even when deployment succeeds.
2. Avoid Redeploy from old deployment history after template updates; use fresh Custom deployment.
3. No hardcoded subscription, resource group, or location values are required.

## Validate Before Deploying

```bash
az deployment group validate \
  --resource-group <RESOURCE-GROUP> \
  --template-file "Watchlist Deployment.json"
```
