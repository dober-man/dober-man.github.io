---
layout: default
title: Configuring a Vulnerable App in Azure with XC - WAS
---

# Configuring a Vulnerable App in Azure with XC Web App Scanner (WAS)

## Overview

F5 offers an advanced Web App Scanner in XC.

A vulnerable web application (Juice Shop) is deployed in Azure and locked down to prevent accidental exposure.  
Network Security Groups (NSGs) are configured to allow only:

- Your source IP  
- XC scanner IP ranges

In this example, a DNS name such as **site1.myfselab.com** is used.

> **Note:** Add any important caveat or prerequisite here.

---

## Juiceshop Deployment Options

You can deploy Juice Shop three different ways:

1. **Azure App Service (recommended):** Clean, repeatable XC WAS/WAF demo. Semi-persistent.  
2. **VM with Docker:** A “mini-lab server” supporting multiple vuln apps. Persistent.  
3. **Azure Container Instances (ACI):** Fast, disposable targets for quick PoCs. Non-persistent.

## Deployment Cost Comparison

Below is a simple comparison of the three deployment methods:

| Deployment Option | Approx. Cost | Persistence | Notes |
|-------------------|-------------|-------------|-------|
| **Azure App Service (B1 Plan)** | ~$54.75/month | **Semi-persistent** | Always billed while the plan exists. Easy HTTPS, easiest demo setup. |
| **Azure VM (1 vCPU / 2GB)** | ~$22–$25/month | **Persistent** | Full control, supports multiple vuln apps, OS must be maintained. |
| **Azure Container Instances (ACI)** | Pay-per-second | **Non-persistent** | Extremely cheap for short-lived demos. Not good for long-running services. |


Choose whichever best matches your demo needs.

---

## Option 1: Deploying via Azure App Service (Recommended)

### Step 1.1 – Create a Resource Group

1. In the Azure Portal, go to **Resource groups → Create**.  
2. Name it **rg-vuln-web-lab**.  
3. Select your subscription and region.  
4. Click **Review + create → Create**.

---

### Step 1.2 – Create the Web App (Docker/Linux)

Navigate to **App Services → Create → Web App**.

#### Basics

- **Subscription:** Your test subscription  
- **Resource Group:** `rg-vuln-web-lab`  
- **Name:** `juiceshop-lab-<unique>`  
- **Publish:** Docker Container  
- **Operating System:** Linux  
- **Region:** Same as RG  
- **Pricing:** B1 or similar (cheap tier works)

#### Docker Settings

- **Options:** Single Container  
- **Image Source:** Docker Hub  
- **Access Type:** Public  
- **Image & Tag:** `bkimminich/juice-shop:latest`

Monitoring: Accept defaults.  
Click **Review + create → Create**.

Azure will provide a URL such as:  
https://juiceshop-lab-<unique>.azurewebsites.net

Visit the URL to confirm the app loads.

---

### Step 1.3 – Lock It Down (Highly Recommended)

In the Web App:

1. Go to **Networking → Access restrictions**.  
2. Add rule:

   - **Name:** allow-my-ip  
   - **Action:** Allow  
   - **Priority:** 100  
   - **IP:** Your public IP (or office/VPN CIDR)

3. Add second rule:

   - **Name:** deny-all  
   - **Action:** Deny  
   - **Priority:** 200  
   - **IP:** `0.0.0.0/0`

This ensures only you (and permitted ranges) can access the vulnerable app.

---

## Option 2: VM Deployment Method (Docker on a Linux VM)

### Step 2.1 – Install Docker on the VM

SSH into your VM and run:

sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

---

### Step 2.2 – Run Juice Shop in Docker

sudo docker run -d --name juiceshop -p 80:3000 bkimminich/juice-shop

Browse to:

http://<VM_PUBLIC_IP>/  
(Assuming your NSG rules only allow your IP.)

---

### Add Other Apps (Optional)

Hackazon Example:

sudo docker run -d --name hackazon -p 8081:80 xex/hackazon

Browse to:

http://<VM_PUBLIC_IP>:8081/  

Useful when testing multiple apps/ports for detection coverage.

---

## Option 3: Azure Container Instances (ACI)

### Azure Container Instances – Quick Disposable Lab

ACI is ideal for quick, single-container, short-lived labs.

### Instructions

Go to **Container Instances → Create**.

### Basics

- **Resource Group:** rg-vuln-web-lab  
- **Container name:** juiceshop-aci  
- **Region:** same as RG  
- **Image source:** Docker Hub  
- **Image type:** Public  
- **Image:** bkimminich/juice-shop  
- **Size:** Small (1 vCPU, 2GB RAM)

### Networking

- **Networking type:** Public  
- **DNS name label:** juiceshop-aci-<unique>  
- **Ports:** 3000  

Browse to:

https://juiceshop-aci-<unique>.<region>.azurecontainer.io:3000

---

## Step 4 – Hooking Your Scanner / WAF / Tools Into This

Once your vulnerable environment is deployed and isolated, connect your scanning and security tools.

### Scanner Targets

- **App Service URL**  
- **VM Public IP or DNS name** on the appropriate port  

### WAF / WAAP / Proxy Workflow (F5 XC, BIG-IP, NGINX, Burp, ZAP)

- Place the WAF or proxy in front of the vulnerable app.  
- Configure the backend/origin to Juice Shop or Hackazon.  
- Run scans through the WAF/proxy path.  
- Observe logs, signature hits, learning events, and blocking behavior.
