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
Sentinel Incident Trigger
│
▼
Get Incident + Extract Entities & Alerts
│
▼
Compose HTML Email Body
│
▼
Check Label (GT_MDE / GT_MDC / GT_MDO / GT_MDI)
│
▼
Switch → Send Email per product route
│ (MDE only)
▼
Advanced Hunting → Server or Client?
├── Server → Send email + Update incident label
└── Client → Send email


## Prerequisites

- Microsoft Sentinel workspace
- Microsoft Defender for Endpoint (MDE) connected to Sentinel
- An Office 365 mailbox for sending notification emails
- Deployment permissions on the target Azure Resource Group

## Deployment

### Deploy via Azure Portal

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/<YOUR_RAW_TEMPLATE_URL>)

> Replace `<YOUR_RAW_TEMPLATE_URL>` with the raw GitHub URL of `Incident-Assignment-Playbook.json`.

### Deploy via Azure CLI

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

### Deploy via PowerShell
New-AzResourceGroupDeployment `
  -ResourceGroupName "<YOUR_RESOURCE_GROUP>" `
  -TemplateFile "Incident-Assignment-Playbook.json" `
  -PlaybookName "Incident-Assignment-Playbook" `
  -NotificationToEmail "soc-team@yourcompany.com" `
  -NotificationFromEmail "sentinel-alerts@yourcompany.com" `
  -ReportName "Your Company SOC" `
  -CompanyLogoLink "https://yourcompany.com/logo.png"

