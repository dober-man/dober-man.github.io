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

1. **Azure App Service (recommended):** Clean, repeatable XC WAS/WAF demo.   
2. **VM with Docker:** A “mini-lab server” supporting multiple vuln apps.   
3. **Azure Container Instances (ACI):** Fast, disposable targets for quick PoCs. 

## Deployment Cost and Timeframe Recommendations

Below is a simple comparison of the three deployment methods:

| Deployment Option | Daily Cost | Ideal Duration | Hard Cutoff | Why Choose It |
|-------------------|-----------|----------------|-------------|----------------|
| **Azure Container Instances (ACI)** | ~$1.30/day | **0–3 days** | **≤ 7 days** | Fast, disposable, no maintenance, cheapest for very short PoCs |
| **Azure App Service (B1 Plan)** | ~$1.83/day | **3–14 days** | **≤ 14 days** | Easiest deployment, built-in HTTPS, great for clean XC demos |
| **Azure VM (1 vCPU / 2GB)** | ~$0.80/day | **14+ days** | Best long-term | Cheapest over time, supports multiple vuln apps, full OS control |


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

### Step 2.1 – Create the VM

1. Go to **Virtual Machines → Create → Azure virtual machine**.  
2. Name: `vuln-web-lab-vm`  
3. Image: Ubuntu 22.04 LTS  
4. Size: B2s  
5. Authentication: SSH key  
6. Networking:  
   - NSG: Only allow **SSH (22)** and **HTTP (80/8081/etc.)** from your IP or scanner IPs.

Create the VM.

---

### Step 2.2 – Install Docker on the VM

SSH into your VM and run:

```bash
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```

Log out and back in (if needed) for group membership to apply.

---

### Step 2.3 – Run Juice Shop in Docker

```bash
sudo docker run -d --name juiceshop -p 80:3000 bkimminich/juice-shop
```

Browse to:

`http://<VM_PUBLIC_IP>/`  
(Assuming your NSG rules only allow your IP.)

---

### Step 2.4 – Lock It Down (Highly Recommended)

On the VM's NIC or subnet NSG:

1. **Inbound rule – allow-my-ip**  
   - Port 80 (and 8081 if using multiple apps)  
   - Source: Your IP or scanner CIDR  
   - Priority: 100  

2. **Deny all**  
   - Port: Any  
   - Source: `0.0.0.0/0`  
   - Priority: 200  

This prevents exposing your vulnerable VM to the entire internet.

---

### Step 2.5 – Add Other Apps (Optional)

Hackazon Example:

```bash
sudo docker run -d --name hackazon -p 8081:80 xex/hackazon
```

Browse to:

`http://<VM_PUBLIC_IP>:8081/`

This setup supports multi-app scanning and multi-port WAF demos.

---

## Option 3: Azure Container Instances (ACI)

### Step 3.1 – Create ACI Container

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

`https://juiceshop-aci-<unique>.<region>.azurecontainer.io:3000`

---

### Step 3.2 – Lock It Down (Highly Recommended)

1. Go to **Networking** → **Firewall** (or use VNet integration).  
2. Restrict inbound access to:  
   - Your IP  
   - XC Scanner ranges  
3. Deny all other inbound sources.

**Note:**  
ACI supports public endpoints by default, so restricting access is critical.

---

## Step 4 – Hooking Your Scanner / WAF / Tools Into This

Once your vulnerable environment is deployed and isolated, connect your scanning and security tools.

### Scanner Targets

- **App Service URL**  
- **VM Public IP or DNS name** on the appropriate port  
- **ACI DNS name**  

### WAF / WAAP / Proxy Workflow (F5 XC, BIG-IP, NGINX, Burp, ZAP)

- Place the WAF or proxy in front of the vulnerable app.  
- Configure the backend/origin to Juice Shop or Hackazon.  
- Run scans through the WAF/proxy path.  
- Observe logs, signature hits, learning events, and blocking behavior.
