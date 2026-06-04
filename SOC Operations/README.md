# PB Enrich Hash/URL Incident Response Playbook

This repository contains an ARM template for a Microsoft Sentinel playbook that enriches file hashes and URLs from incident entities, then performs containment when malicious indicators are confirmed.

Playbook flow:

1. Extract file hash, URL, host, and account entities from the Sentinel incident.
2. Enrich file hashes with VirusTotal.
3. Enrich URLs with PhishTank.
4. If malicious hash or URL is found:
5. Systemic containment: hard delete email via Defender API.
6. Host containment: isolate each matched machine.
7. User containment: revoke sessions and force password change on next sign-in.
8. Add enrichment summary comment to the incident.

## Template File

Use this template:

1. PB-Enrich-HashUrl-IncidentResponse.json

## Parameters

| Parameter | Required | Default | Description |
|---|---|---|---|
| workflows_PB_Enrich_HashUrl_IncidentResponse_name | No | PB-Enrich-HashUrl-IncidentResponse | Logic App workflow name |
| location | No | Resource group location | Deployment region |
| azuresentinelConnectionName | No | azuresentinel-incidentresponse | Sentinel API connection name to create/use |
| azuremonitorlogsConnectionName | No | azuremonitorlogs-2 | Azure Monitor Logs API connection name to create/use |
| connections_azuresentinel_2_externalid | No | Dynamic resourceId(...) | Sentinel API connection resource ID |
| connections_azuremonitorlogs_1_externalid | No | Dynamic resourceId(...) | Azure Monitor Logs API connection resource ID |

Runtime workflow parameters:

1. VTApiKey (required at deployment/runtime)
2. PhishTankAppKey (required at deployment/runtime)
3. MaliciousThreshold (default 1)

## What The Template Deploys

1. Microsoft.Web/connections for Sentinel
2. Microsoft.Web/connections for Azure Monitor Logs
3. Microsoft.Logic/workflows (SystemAssigned managed identity)

The workflow dependsOn both connection resources so they are created first.

## Prerequisites

1. Microsoft Sentinel enabled on target workspace.
2. Defender XDR / Defender for Endpoint data integrated with Sentinel incidents.
3. Permission to deploy Logic Apps and API connections in target resource group.
4. Ability to authorize API connections after deployment.
5. VirusTotal and PhishTank API credentials available.

## Permissions Required

### A. Deployment Identity Permissions

1. Contributor (or equivalent) on target resource group.
2. Permission to create and edit Microsoft.Web/connections.

### B. API Connection Authorization Account Permissions

1. Sentinel data connector access to authorize azuresentinel API connection.
2. Log Analytics query access to authorize azuremonitorlogs API connection.

### C. Logic App Managed Identity Permissions (Runtime)

The workflow uses managed identity for HTTP actions to Defender and Microsoft Graph.

Required capabilities:

1. Defender incidents read.
2. Defender machines read.
3. Defender machine isolation action.
4. Defender email remediation (HardDelete).
5. Microsoft Graph user session revocation.
6. Microsoft Graph user profile update for forceChangePasswordNextSignIn.

Grant corresponding app permissions and admin consent in tenant where needed.

## Deploy

### Azure Portal

1. Open Custom deployment in target resource group.
2. Upload PB-Enrich-HashUrl-IncidentResponse.json.
3. Provide parameter values if overriding defaults.
4. Deploy.

### Azure CLI

```bash
az deployment group create \
  --resource-group <RESOURCE-GROUP> \
  --template-file PB-Enrich-HashUrl-IncidentResponse.json
```

## Post-Deployment Steps

### 1. Authorize Connections

1. Go to Resource Group -> API connections.
2. Open azuresentinel-incidentresponse (or your chosen Sentinel connection name).
3. Select Edit API connection -> Authorize -> Save.
4. Open azuremonitorlogs-2 (or your chosen Logs connection name).
5. Select Edit API connection -> Authorize -> Save.

### 2. Enable and Trigger

1. Confirm Logic App is enabled.
2. Attach playbook to Sentinel automation rule.
3. Test with incident containing hash or URL entities.

## Notes

1. If deployment is run from old deployment history, stale parameter values can be reused.
2. Prefer fresh Custom deployment after template changes.
3. If API connections are created but not authorized, workflow trigger/actions will fail at runtime.

## Validate Before Deploying

```bash
az deployment group validate \
  --resource-group <RESOURCE-GROUP> \
  --template-file PB-Enrich-HashUrl-IncidentResponse.json
```
