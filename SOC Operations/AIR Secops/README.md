AIR-SECOPS-Tagging-Medium-High-Severity

📌 Description
This playbook automates Microsoft Sentinel incident tagging based on Microsoft Defender investigation states identified from associated alerts.
The workflow evaluates investigation-related properties from the SecurityAlert table and automatically adds the“Secops” tag when analyst or SecOps intervention may be required. [microsofta...epoint.com]

⚙️ Features
Automatically triggers on new Microsoft Sentinel incidents
Retrieves associated incident alerts and processes them sequentially
Executes KQL queries against the SecurityAlert table in Log Analytics
Evaluates Microsoft Defender investigation states from ExtendedProperties 
Automatically adds the“Secops” tag to qualifying incidents
Uses Managed Identity authentication for Azure Sentinel connection


🔄 Workflow Overview

Triggered when a Microsoft Sentinel incident is created 
Retrieves associated alerts from the incident payload 
Extracts SystemAlertId for each alert 
Executes KQL query against SecurityAlert table 
Reads investigation states from:

MicrosoftDefenderAtp.InvestigationState
Status 
Checks whether the alert matches supported investigation states
If condition is true:
Adds "Secops" tag to incident


✅ Supported Investigation States
The playbook adds the“Secops” tag when alerts contain any of the following states:

TerminatedBySystem
TerminatedByUser
Failed
PartiallyRemediated
PendingResource
PendingApproval
PendingAction
Pending

📂 Files Included

AIR Secops Playbook.json → ARM template for Logic App deployment

🚀 Deployment Steps
Option 1: Azure Portal

Go to Azure Portal
Navigate to Deploy a custom template
Upload AIR Secops Playbook.json
Provide deployment parameter values
Deploy the template

🔐 Permissions Required
After deployment, assign the following roles to the Logic App Managed Identity:

Microsoft Sentinel Contributor / Responder
Log Analytics Reader 

Required permission scope

Microsoft Sentinel Workspace
Log Analytics Workspace


⚠️ Important Notes

The playbook runs only when the condition evaluates to true 
Incident tags are updated by appending "Secops" 
Workflow concurrency is configured as sequential (repetitions = 1) 
KQL query execution uses a 24 hour time range
Managed Identity authentication is used for Azure Sentinel connection


💡 Use Cases

AIR-driven incident routing
SOC incident enrichment
Automated SecOps tagging
Analyst escalation workflows
Microsoft Defender alert investigation handling


🛠️ Customization
You can modify:

Investigation states in KQL logic
Tag value (Secops)
Trigger conditions
Query time range
Concurrency settings
