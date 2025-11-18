--- 
layout: default
title: Configuring a Vulnerable App in Azure with XC - WAS
---

# Deploying Vulnerable Web Apps in Azure With Automated Expiration Cleanup

## Overview

This guide helps you deploy intentionally vulnerable web applications (such as OWASP Juice Shop, Hackazon, DVWA, etc.) into Azure **safely**, **locked down**, and with **automatic cleanup** using an Azure Logic App. 

This prevents:

- Accidental exposure of vulnerable apps  
- Lingering cloud resources that continue to incur costs  

The Logic App will **delete resources when their expiration tag is reached**, ensuring that your lab environments remain temporary and controlled. The Logic App costs ~$0.001 – $0.02 per month.

---

# Prerequisites

Before deploying any vulnerable applications, you must configure:

1. **A Resource Group** where the apps will live  
2. **A Logic App** that automatically deletes expired resources  
3. **Tagging standards** so each deployment includes an expiration timestamp  

---

# Step A — Create the Resource Group
1. Go to **Resource Groups → Create**  
2. Name: `rg-vuln-web-lab`  
3. Region: your preferred region  
4. Create

---

## Step A.1 — Configure the Logic App (Expiration-Based Cleanup)

This Logic App periodically checks resources for an `expireOn` tag and automatically deletes them after the expiration date/time.

---

## Step A.2 — Create the Logic App

1. Go to **Azure Portal → Logic Apps → Create**  
2. Choose **Consumption** (recommended for PoCs)  
3. Use:
   - **Resource group:** `rg-vuln-web-lab`
   - **Name:** `vuln-lab-expiration-cleanup`
   - **Region:** same as your lab resources  
4. Click **Review + create → Create**

---

## Step A.3 — Build the Logic App Workflow

Open the **Logic App Designer** and follow the steps below.

---

### 1. Trigger: **Schedule → Recurrence**

Configure the trigger:

- **Frequency:** Day  
- **Interval:** 1  

<img src="/xc-images/logicapp1.png" style="max-width:600px; width:100%; height:auto;">

---

### 2. Add Action: **List Resources (By Resource Group)**

Add the action:

**Azure Resource Manager → List resources (Resource Group)**

<img src="/xc-images/listresource1.png" style="max-width:600px; width:100%; height:auto;">

This retrieves all resources inside **rg-vuln-web-lab** so the Logic App can inspect each resource’s tags.

---

### 3. Add Connection (Authentication)

When prompted to create a connection, select:

- **Authentication:** Managed identity (recommended)

This ensures the Logic App uses its own identity to access and delete resources in the scoped Resource Group.

<img src="/xc-images/connect.png" style="max-width:600px; width:100%; height:auto;">

#### Other Authentication Options (When to Use / Avoid)

- **OAuth:**  
  Uses *your* user account. Good for quick testing but not ideal for long-term or least-privilege automation.

- **Service Principal:**  
  Useful for large enterprise deployments where app registrations and secrets are standardized. Overkill for a small lab.

---

### 4. Configure Managed Identity (Recommended)

#### **A. Enable the Logic App’s Managed Identity**

1. Open your Logic App  
2. Go to **Identity** (left menu)  
3. Under **System assigned**, toggle **On**  
4. Click **Save**

#### **B. Grant RBAC Permissions on the Resource Group**

1. Go to **Resource groups → rg-vuln-web-lab**  
2. Open **Access control (IAM)**  
3. Select **Add role assignment**  
4. Choose **Contributor** (required for delete operations)  
5. Assign access to: **Managed Identity**  
6. Select your Logic App → Save

#### **C. Complete the Connection Setup**

Back in the “Create connection” dialog:

- **Authentication:** Managed identity  
- **Managed identity type:** System-assigned  
- **Subscription:** Your subscription  
- **Resource Group:** `rg-vuln-web-lab`

---

### 5. **For Each Resource**
Add **For Each** and loop over the resource list.

### 6. **Condition: Check for expireOn tag**

```
@if(lessOrEquals(item()?['tags']?['expireOn'], utcNow()), true, false)
```

### 7. **If TRUE → Delete Resource**
Add:

**Azure Resource Manager → Delete resource**

Use the resource ID from the loop item.

### 8. (Optional) **Send Notification**
- Email  
- Teams  
- Webhook  

---

# Tagging Standard for All Deployments

| Tag Key     | Purpose |
|-------------|---------|
| **expireOn** | UTC timestamp when cleanup should occur |
| **owner** | Useful for multi-user labs |
| **demo** | Identifies purpose (ex: juiceshop-aci) |

### Example Tag Format

```
expireOn = 2025-11-25T23:59:00Z
owner    = mike
demo     = juiceshop-lab
```

---

# Deployment Options for Juice Shop

You can deploy Juice Shop three different ways:

1. **Azure App Service (Recommended)**  
2. **Azure VM + Docker**  
3. **Azure Container Instances (ACI)**  

---

# Option 1 — Azure App Service (Recommended)

### Recommended Tag Example
```
expireOn = 2025-11-25T23:59:00Z
demo     = juice-appservice
```

## Step 1.2 — Deploy the App Service

Navigate to **App Services → Create → Web App**

### Basics
- Publish: **Docker Container**  
- OS: **Linux**  
- Plan: **B1**  
- Image: `bkimminich/juice-shop:latest`  
- Region: Same as RG  

Verify:

```
https://juiceshop-lab-<unique>.azurewebsites.net
```

---

## Step 1.3 — Lock It Down

**Web App → Networking → Access Restrictions**

Add:

1. **allow-my-ip**
2. **allow-scanner-ips** (optional)
3. **deny-all** (0.0.0.0/0)

---

## Step 1.4 — Add Expiration Tags

```
expireOn = <UTC timestamp>
owner    = mike
demo     = juice-appservice
```

---

# Option 2 — Azure VM (Docker)

### Recommended Tag Example
```
expireOn = 2025-11-25T23:59:00Z
demo     = juice-vm
```

## Step 2.1 — Create the VM
- Ubuntu 22.04  
- Size: B2s  
- NSG inbound rules: allow only your IP + deny-all  

---

## Step 2.2 — Install Docker

```
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```

---

## Step 2.3 — Run Juice Shop

```
sudo docker run -d --name juiceshop -p 80:3000 bkimminich/juice-shop
```

Browse:

```
http://<VM_PUBLIC_IP>/
```

---

## Step 2.4 — Add Expiration Tags

```
expireOn = <UTC timestamp>
owner    = mike
demo     = juice-vm
```

---

# Option 3 — Azure Container Instances (ACI)

### Recommended Tag Example
```
expireOn = 2025-11-25T23:59:00Z
demo     = juice-aci
```

## Step 3.1 — Create ACI

- Image: `bkimminich/juice-shop`  
- 1 vCPU / 2 GB  
- Port: 3000  

URL:

```
https://juiceshop-aci-<unique>.<region>.azurecontainer.io:3000
```

---

## Step 3.2 — Lock It Down

Use ACI Firewall or VNet + NSG.

---

## Step 3.3 — Add Expiration Tags

```
expireOn = <UTC timestamp>
owner    = mike
demo     = juice-aci
```

---

# Cleanup Happens Automatically

When the expiration timestamp is reached:

- Logic App detects expired resources  
- Deletes them  
- (Optional) Sends notification  

Your environment stays clean and cost-effective.

