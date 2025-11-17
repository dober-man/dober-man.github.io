Configuring a Vulnerable App in Azure with XC - WAS

Configuring a Vulnerable App in Azure with XC Web App Scanner (WAS)

Overview

F5 offers an advanced Web App Scanner in XC.

A vulnerable web application (Juice Shop) is deployed in Azure and
locked down to prevent accidental exposure.
Network Security Groups (NSGs) are configured to allow only:

-   Your source IP
-   XC scanner IP ranges

In this example, a DNS name such as site1.myfselab.com is used.

  Note: Add any important caveat or prerequisite here.

------------------------------------------------------------------------

Azure Configuration

Step 1.1 – Create a Resource Group

1.  In the Azure Portal, go to Resource groups → Create.
2.  Name it rg-vuln-web-lab.
3.  Select your subscription and region.
4.  Click Review + create → Create.

------------------------------------------------------------------------

Step 1.2 – Create the Web App (Docker/Linux)

Navigate to App Services → Create → Web App.

Basics

-   Subscription: Your test subscription
-   Resource Group: rg-vuln-web-lab
-   Name: juiceshop-lab-<unique>
-   Publish: Docker Container
-   Operating System: Linux
-   Region: Same as RG
-   Pricing: B1 or similar (cheap tier works)

Docker Settings

-   Options: Single Container
-   Image Source: Docker Hub
-   Access Type: Public
-   Image & Tag: bkimminich/juice-shop:latest

Monitoring: Accept defaults.
Click Review + create → Create.

Azure will provide a URL such as:
https://juiceshop-lab-.azurewebsites.net

Visit the URL to confirm the app loads.

------------------------------------------------------------------------

Step 1.3 – Lock It Down (Highly Recommended)

In the Web App:

1.  Go to Networking → Access restrictions.

2.  Add rule:

    -   Name: allow-my-ip
    -   Action: Allow
    -   Priority: 100
    -   IP: Your public IP (or office/VPN CIDR)

3.  Add another rule:

    -   Name: deny-all
    -   Action: Deny
    -   Priority: 200
    -   IP: 0.0.0.0/0

This ensures only you (and permitted ranges) can access the vulnerable
app.

------------------------------------------------------------------------

Optional: VM Deployment Method

Step 2.1 – Install Docker on the VM

SSH into your VM and run:

    sudo apt-get update
    sudo apt-get install -y docker.io
    sudo systemctl enable docker
    sudo systemctl start docker
    sudo usermod -aG docker $USER

Step 2.2 – Run Juice Shop in Docker

To launch Juice Shop on your VM:

    sudo docker run -d --name juiceshop -p 80:3000 bkimminich/juice-shop

Once running, browse to:

http:///

(Assuming your NSG rules only allow your IP.)

Add Other Apps (Optional)

Hackazon Example

Deploy Hackazon using a public Docker image:

    sudo docker run -d --name hackazon -p 8081:80 xex/hackazon

Access it at:

http://:8081/

You can direct scanners at additional ports/apps to test detection and
coverage.

Optional Deployment Method – Azure Container Instances (ACI)

Azure Container Instances – Quick Disposable Lab

Azure Container Instances provide a quick, disposable lab.

Instructions

Go to Container Instances → Create.

Basics

-   Resource Group: rg-vuln-web-lab
-   Container name: juiceshop-aci
-   Region: same as RG
-   Image source: Docker Hub
-   Image type: Public
-   Image: bkimminich/juice-shop
-   Size: Small (1 vCPU, 2GB RAM)

Networking

-   Networking type: Public
-   DNS name label: juiceshop-aci-
-   Ports: 3000

Browse to:

https://juiceshop-aci-..azurecontainer.io:3000

------------------------------------------------------------------------

Step 4 – Hooking Your Scanner / WAF / Tools Into This

Once your vulnerable environment is deployed and isolated, connect your
scanning and security tools.

Scanner Targets

-   App Service URL
-   VM Public IP or DNS name on the appropriate port

WAF / WAAP / Proxy Workflow (F5 XC, BIG-IP, NGINX, Burp, ZAP)

-   Place the WAF or proxy in front of the vulnerable app.
-   Configure the backend/origin to Juice Shop or Hackazon.
-   Run scans through the WAF/proxy path.
-   Observe logs, signature hits, learning events, and blocking
    behavior.
