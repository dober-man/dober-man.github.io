---
layout: default
title: Configuring a Vulnerable App in Azure with XC - WAS
---

# Deploying Vulnerable Web Apps in Azure With Automated Expiration Cleanup

## Overview

This guide helps you deploy intentionally vulnerable web applications (such as OWASP Juice Shop, Hackazon, DVWA, etc.) into Azure **safely**, **locked down**, and with **automatic cleanup** using an Azure Logic App. Cost ~$0.001 – $0.02 per month

This prevents:

- Accidental exposure of vulnerable apps  
- Lingering cloud resources that continue to incur costs  

The Logic App will **delete resources when their expiration tag is reached**, ensuring that your lab environments remain temporary and controlled. 

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

## Step A.1 — Configure the Logic App (Expiration-Based Cleanup)

This Logic App periodically checks resources for an `expireOn` tag and automatically deletes them after the expiration date/time.

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

Open the **Logic App Designer** and add:

### 1. **Trigger: Schedule** -> **Recurrence**

- Frequency: `1`  
- Interval: `Day` (or Hour for short-lived demos)


    ![Logic App Trigger](/xc-images/logicapp.png)


### 2. **List Resources**
Add action:  
**Azure Resource Manager → List resources (Subscription)**

   ![List Resources](/xc-images/listresource.png)

This fetches all Azure resources so the Logic App can inspect their tags.

### 3. **For Each Resource**
Add **For Each** and loop over the resource list.

Inside the loop:

### 4. **Condition: Check for expireOn tag**
Use an expression such as:

```
@if(lessOrEquals(item()?['tags']?['expireOn'], utcNow()), true, false)
```

### 5. **If TRUE → Delete Resource**
Add:

**Azure Resource Manager → Delete resource**

Use the resource ID from the loop item.

### 6. (Optional) **Send Notification**
Actions you can attach:

- Send email  
- Teams notification  
- Webhook to monitoring system  

---

# Tagging Standard for All Deployments

Every vulnerable environment must include:

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

Each option below includes where to set your expiration tags.

---

# Option 1 — Azure App Service (Recommended)

### Recommended Tag Example
```
expireOn = 2025-11-25T23:59:00Z
demo     = juice-appservice
```

---

## Step 1.2 — Deploy the App Service

Navigate to **App Services → Create → Web App**

### Basics
- Publish: **Docker Container**  
- OS: **Linux**  
- Plan: **B1**  
- Image: `bkimminich/juice-shop:latest`  
- Region: Same as RG  

Deploy and verify:

```
https://juiceshop-lab-<unique>.azurewebsites.net
```

---

## Step 1.3 — Lock It Down

Go to:

**Web App → Networking → Access Restrictions**

Add:

1. **allow-my-ip**
2. **allow-scanner-ips** (if applicable)
3. **deny-all** (0.0.0.0/0)

---

## Step 1.4 — Add Expiration Tags

Go to:

**Web App → Settings → Configuration → Tags**

Add:

```
expireOn = <UTC timestamp>
owner    = mike
demo     = juice-appservice
```

---

# Option 2 — Deploy on Azure VM (Docker)

### Recommended Tag Example
```
expireOn = 2025-11-25T23:59:00Z
demo     = juice-vm
```

## Step 2.1 — Create the VM
- Ubuntu 22.04  
- Size: B2s  
- NSG inbound rules: allow only  
  - your IP  
  - scanner IPs  
  - deny-all for rest  

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

## Step 2.4 — Lock It Down with NSG
- allow-my-ip  
- allow-scanner-ips  
- deny-all  

---

## Step 2.5 — Add Expiration Tags

Go to:

**VM → Tags**

Add:

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
- CPU/RAM: 1 vCPU / 2 GB  
- Networking: Public  
- Port: 3000  

The URL will look like:

```
https://juiceshop-aci-<unique>.<region>.azurecontainer.io:3000
```

---

## Step 3.2 — Lock It Down

Use ACI Firewall or attach to a VNet with NSG:

- allow-my-ip  
- allow-scanner-ips  
- deny-all  

---

## Step 3.3 — Add Expiration Tags

Go to:

**Container Instance → Tags**

Add:

```
expireOn = <UTC timestamp>
owner    = mike
demo     = juice-aci
```

---

# Step 4 — (Optional) Integrate With Scanners / WAF

Once your apps are deployed and locked down, you may route them through:

- F5 XC WAAP  
- BIG-IP WAF  
- NGINX App Protect  
- Burp/ZAP  
- Custom scanners  

But this is **separate** from the Logic App cleanup process.

---

# Cleanup Happens Automatically

When the expiration timestamp is reached:

- Logic App detects expired resources  
- Deletes them  
- (Optional) Sends notification  

Your environment stays clean and cost-effective.

