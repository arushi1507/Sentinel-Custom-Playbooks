# Entra Risky Sign-In Containment Playbook

This repository contains an ARM template that deploys a Microsoft Sentinel playbook for risky sign-in containment in Microsoft Entra ID.

The playbook does the following when triggered by a Sentinel incident:

1. Extracts account entities from the incident.
2. Iterates user accounts and applies containment actions with Microsoft Graph.
3. Revokes active sign-in sessions.
4. Forces password reset at next sign-in with MFA.
5. Attempts manager notification email.
6. Optionally posts an update to a Teams webhook.

## Template File

Use this file for deployment:

- Entra-RiskySignIn-Containment-Prod-Playbook.json

## What This Template Deploys

The template creates both resources in one deployment:

1. API connection resource: Microsoft.Web/connections
2. Logic App playbook: Microsoft.Logic/workflows (system-assigned identity enabled)

## Parameters

| Parameter | Required | Default | Description |
|---|---|---|---|
| workflowName | No | Entra-RiskySignIn-Containment-Playbook | Logic App name |
| location | No | Resource group location | Azure region |
| connectionName | No | azuresentinel-Conn | Name for Azure Sentinel API connection |

## Prerequisites

1. Microsoft Sentinel enabled on your Log Analytics workspace.
2. Deployment permissions on target resource group.
3. Permission to authorize API connections in the target resource group.
4. Graph permissions for the playbook managed identity to perform user containment operations.

## Deploy

### Azure Portal

1. Open custom deployment in the target resource group.
2. Upload Entra-RiskySignIn-Containment-Prod-Playbook.json.
3. Provide parameter values if needed.
4. Run deployment.

### Azure CLI

```bash
az deployment group create \
  --resource-group <RESOURCE-GROUP> \
  --template-file Entra-RiskySignIn-Containment-Prod-Playbook.json
```

## Required Post-Deployment Steps

### 1. Authorize API Connection

After deployment, authorize the created API connection:

1. Go to Resource group -> API connections.
2. Open the connection named by connectionName (default: azuresentinel-Conn).
3. Select Edit API connection.
4. Select Authorize and sign in.
5. Save.

If this step is skipped, the workflow trigger/actions using Sentinel connector will fail.

### 2. Configure Managed Identity Permissions for Graph

The playbook uses system-assigned managed identity for Microsoft Graph HTTP actions.

Grant appropriate Graph application permissions for your containment policy, then grant admin consent.

Minimum permissions depend on your tenant policy and chosen actions. Common examples:

1. User.RevokeSessions.All
2. User.ReadWrite.All
3. Mail.Send (if manager notification is used)

### 3. Enable the Playbook

The template deploys the workflow in Disabled state by default.

1. Open the deployed Logic App.
2. Enable the workflow.
3. Attach it to a Sentinel automation rule as needed.

## Notes

1. The template is sanitized for GitHub sharing: no hardcoded tenant-specific subscription IDs, resource groups, or MPG tags.
2. The API connection is created by the same template to avoid missing-connection deployment errors.
3. If you redeploy from Azure Deployment History, old parameter values may be reused. Use a fresh custom deployment when testing template changes.

## Validation (Optional)

```bash
az deployment group validate \
  --resource-group <RESOURCE-GROUP> \
  --template-file Entra-RiskySignIn-Containment-Prod-Playbook.json
```
