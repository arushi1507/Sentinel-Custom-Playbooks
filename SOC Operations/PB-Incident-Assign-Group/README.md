# Incident-Assignment-Playbook

A Microsoft Sentinel Logic App playbook that automatically routes incident notification emails based on the **incident label** (tag) assigned in Sentinel. It supports routing for Microsoft Defender for Endpoint (MDE), Defender for Cloud (MDC), Defender for Office 365 (MDO), and Defender for Identity (MDI) incidents.

## Overview

When a new Microsoft Sentinel incident is created, this playbook:

1. Reads the incident labels to determine the product source (`GT_MDE`, `GT_MDC`, `GT_MDO`, `GT_MDI`)
2. Composes an HTML incident report email with incident details, entities, and alerts
3. Routes and sends the email notification to the configured recipients
4. For **MDE incidents** — runs an Advanced Hunting query in Microsoft Defender for Endpoint to check if the affected device is a **Server** or **Client**, and:
   - If Server: sends the email and updates the incident label from `GT_MDE` → `GT_MDC`
   - If Client: sends the email as a standard MDE alert

## Architecture

Microsoft Sentinel Incident Trigger
    ↓
Get Incident + Extract Entities & Alerts
    ↓
Compose HTML Incident Email
    ↓
Check Incident Label
    ↓
Switch Routing Logic
    ↓
Advanced Hunting Query (MDE only)
    ↓
Server or Client Decision
    ↓
Send Email Notification

## Prerequisites

- Microsoft Sentinel enabled workspace
- Microsoft Defender for Endpoint integrated with Sentinel
- Office 365 mailbox for email notifications
- Contributor access to Azure Resource Group

## Parameters

- PlaybookName
- location
- connections_azuresentinel_name
- connections_office365_name
- connections_wdatp_name
- NotificationToEmail
- NotificationFromEmail
- ReportName
- CompanyLogoLink

## Post-Deployment Steps

1. Authorize Office 365 API Connection
2. Assign Sentinel Responder role to Managed Identity
3. Configure Sentinel Automation Rule
4. Apply Sentinel Labels for routing

## Incident Labels

GT_MDE → Microsoft Defender for Endpoint
GT_MDC → Microsoft Defender for Cloud
GT_MDO → Microsoft Defender for Office 365
GT_MDI → Microsoft Defender for Identity

## Deployment

### Deploy via Azure Portal

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/<YOUR_RAW_TEMPLATE_URL>)

> Replace `<YOUR_RAW_TEMPLATE_URL>` with the raw GitHub URL of `Incident-Assignment-Playbook.json`.

## Deployment via Azure CLI

```bash
az deployment group create \
  --resource-group <YOUR_RESOURCE_GROUP> \
  --template-file Incident-Assignment-Playbook.json \
  --parameters \
      PlaybookName="Incident-Assignment-Playbook" \
      NotificationToEmail="soc-team@yourcompany.com" \
      NotificationFromEmail="sentinel-alerts@yourcompany.com" \
      ReportName="Your Company SOC" \
      CompanyLogoLink="https://yourcompany.com/logo.png"

