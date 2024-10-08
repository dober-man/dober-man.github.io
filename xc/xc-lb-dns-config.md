---
layout: default
title: Configuring DNS for XC Load Balancer
---

# Configuring DNS Delegation to XC. 

There are 3 options for getting clients resolved appropriately to the XC Cloud Load Balancer.

1. "A" record Delegation
2. CNAME Delegation
3. Subdomain Delegation

## "A" record delegation 
While not a true delegation so to speak, the "A" record delegation is quick and straightforward to configure. 

> **Note:** Auto Certificate management not supported in this configuration. BYOC.  

1. Create an XC-LB
2. Retrieve XC public IP
3. Modify existing sites "A" record to point to XC IP. 




## CNAME Delegation Example - uses GoDaddy
1. Create an XC-LB
2. Retrieve XC LB CNAME
3. Modify existing sites "A" record to become a CNAME that resolves the site to an XC CNAME and IP. 






## Subdomain Delegation Example - uses GoDaddy

  * Add subdomain to be delegated as a primary domain in XC DNS.
    * Make sure to check the box for: "Allow Application Load Balancer Managed Records" under the Primary DNS Configuration options.
  * In this example the test subdomain to be delegated is: **mytest**.myfselab.us

2. Add NS servers with a delegation to **mytest** to customer managed DNS Servers (In this example the DNS provider is GoDaddy.

    ![GoDaddy DNS config](../images/ns.png)

3. Create LB object with auto-cert enabled. EX: site1.mytest.myfselab.us




