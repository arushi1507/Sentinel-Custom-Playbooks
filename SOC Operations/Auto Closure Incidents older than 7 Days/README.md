# PB-Incident-Close-Old-Incidents

## 📌 Description

This playbook automates the closure of incidents in Microsoft Sentinel that are older than a defined threshold (default: 7 days).

It helps SOC teams reduce noise by automatically closing stale incidents that were not manually handled.

---

## ⚙️ Features

* Automatically identifies incidents older than a configurable number of days
* Supports **dry run mode** (no actual closure, only simulation)
* Uses **Managed Identity** for secure API access
* Scalable with configurable batch size (`topToFetch`)
* Runs on a daily schedule

---

## 🔄 Workflow Overview

1. Triggered on a daily schedule
2. Calculates cutoff date (current date - defined threshold)
3. Fetches incidents from Sentinel using REST API
4. Filters incidents:

   * Older than threshold
   * Status not equal to "Closed"
5. For each incident:

   * If `dryRun = true` → logs incident ID
   * If `dryRun = false` → closes incident

---

## 📂 Files Included

* `azuredeploy.json` → ARM template for deployment
* `parameters.json` → Input parameters for deployment

---

## 🚀 Deployment Steps

### Option 1: Azure Portal

1. Go to Azure Portal
2. Navigate to **Deploy a custom template**
3. Upload `azuredeploy.json`
4. Provide values from `parameters.json`
5. Deploy

---

### Option 2: Azure CLI

```bash
az deployment group create \
  --resource-group <RESOURCE_GROUP> \
  --template-file azuredeploy.json \
  --parameters @parameters.json
```

---

## 🧩 Required Parameters

| Parameter               | Description                           |
| ----------------------- | ------------------------------------- |
| workspaceSubscriptionId | Subscription ID of Sentinel workspace |
| workspaceResourceGroup  | Resource group of workspace           |
| workspaceName           | Log Analytics workspace name          |

---

## 🔐 Permissions Required

After deployment, assign the following role to the Logic App Managed Identity:

* **Microsoft Sentinel Responder**

Required permission scope:

* Log Analytics Workspace or Resource Group

---

## ⚠️ Important Notes

* Default `dryRun = true` → No incidents will be closed initially
* Change `dryRun` to `false` after validation
* Ensure API version compatibility with your environment
* Validate in test environment before production use

---

## 💡 Use Cases

* Auto-close stale incidents
* Reduce SOC backlog
* Improve incident hygiene
* Support automated SOC operations

---

## 🛠️ Customization

You can modify:

* `olderThanDays` → Change threshold
* `topToFetch` → Control batch size
* Recurrence trigger → Adjust schedule

---

## 👤 Author

SOC Automation

---

## 📄 License

This project is intended for internal SOC automation and reusable playbook deployment.

