---
layout: default
title: Configuring a Vulnerable App in Azure with XC - WAS
---

## Overview

F5 offers an advanced Web App Scanner in XC. 
Pricing: 

A vulnerable webapp (Juiceshop) is installed in Azure and locked down to prevent accidental exposure. 
NSG's are confugred to only allow my source IP and the XC scanner IP's 

* In this example a site called site1.myfselab.com is used. 

**Note:** Something

## Azure Config

### Step 1.1 – Create a Resource Group

In the Azure Portal: Resource groups → Create Name **rg-vuln-web-lab**
* Choose your subscription + region.
* Click Review + create → Create.

### Step 1.2 – Create the Web App (Docker/Linux)

App Services → Create → Web App.

#### Basics:

Subscription: your test sub

Resource Group: rg-vuln-web-lab

Name: e.g. juiceshop-lab-<unique>

Publish: Docker Container

Operating System: Linux

Region: same as RG

Pricing: a cheap tier is fine (B1 or similar)

Docker tab:

Options: Single Container

Image Source: Docker Hub

Access Type: Public

Image and tag:
bkimminich/juice-shop:latest

Monitoring: you can leave defaults.

Review + create → Create.

Once deployed, Azure will give you a URL like:

https://juiceshop-lab-<something>.azurewebsites.net

Visit it in a browser to confirm it loads.

###  Step 1.3 – Lock it down (super important)

From the Web App:

Go to Networking → Access restrictions.

Add a rule:

Name: allow-my-ip

Action: Allow

Priority: 100

IP: your public IP (or your office / VPN IP range).

Add a second rule:

Name: deny-all

Action: Deny

Priority: 200

IP: 0.0.0.0/0

Now only you (and your chosen ranges) can hit the vuln app.

### Step 2.1 – Install Docker on the VM

SSH into the VM:

# Update
sudo apt-get update

# Install Docker
sudo apt-get install -y docker.io

# Enable & start
sudo systemctl enable docker
sudo systemctl start docker

# Add your user to docker group (optional)
sudo usermod -aG docker $USER


### Step 2.2 – Run Juice Shop in Docker


sudo docker run -d --name juiceshop -p 80:3000 bkimminich/juice-shop

Now http://<VM_PUBLIC_IP>/ should show Juice Shop (again, hopefully only reachable from your IP due to NSG rules).

### Step 2.3 – Add other apps (optional)

Hackazon: (one simple approach – there are several images out there):
sudo docker run -d --name hackazon -p 8081:80 xex/hackazon

Hit it at: http://<VM_PUBLIC_IP>:8081/

You can then point your scanner at specific ports/apps to test coverage and detection.

## Optional Deployment Method (to test)
Option C: Azure Container Instances (ACI) for quick, throwaway labs

If you don’t want a full VM:

Container instances → Create.

Basics:

Resource Group: rg-vuln-web-lab

Container name: juiceshop-aci

Region: same as RG

Image source: Docker Hub

Image type: Public

Image: bkimminich/juice-shop

Size: small (1 vCPU, 2GB RAM is plenty).

Networking:

Networking type: Public

DNS name label: juiceshop-aci-<unique>

Ports: 3000

Review + create → Create.

Browse to: http://juiceshop-aci-<unique>.<region>.azurecontainer.io:3000.

Use NSG / firewall rules or restrict at the VNet level if you use Private networking instead.


## Step 4 Hooking your scanner / WAF / tool into this

Once your broken app is up and isolated:

Point your web app scanner at:

the App Service URL, or

the VM public IP / DNS name + port.

If you’re testing WAAP / WAF / proxy (like F5 XC, Burp, ZAP, etc.):

Put the proxy/WAF in front of the vulnerable app.

Configure the backend (origin) to point to the Juice Shop / Hackazon endpoint.

Run scans through that path and watch logs, signatures, learning, etc.






