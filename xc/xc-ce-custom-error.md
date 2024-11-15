---
layout: default
title: Configuring a Custom Response Page on CE
---

## Overview

* By default the CE does not respond with a default error page. 

* LB Config regarding Serviec policy response page is irrelevant (only applies to RE)


1.  Add subdomain to be delegated **site1.myfselab.com** as a primary domain in XC DNS.

  ![New Zone](/xc-images/zone.png)

* Make sure to check the box for: "Allow Application Load Balancer Managed Records" under the Primary DNS Configuration options.

    ![Auto DNS for LB](/xc-images/lbr.png)

2. In GoDaddy, add F5 Cloud DNS Server NS records with a delegation to **site1**.

    ![GoDaddy DNS config](/xc-images/f5ns.png)

3. Create LB object with auto-cert enabled. EX: site1.mytest.myfselab.us

    ![HTTP LB config](/xc-images/lb.png)

4. Verify - The cert generation process can take a few minutes. You will see it in a pending state. Click on the "i" for more info. 

    ![Pending Challenge](/xc-images/pending.png)

5. Success

    ![Success](/xc-images/success.png)

