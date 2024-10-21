---
layout: default
title: Preventing XC to XC Malicious Activity
---

## Overview

F5 Distributed Cloud offers a suite of powerful features designed to simplify the lives of administrators and engineers. A key aspect of this ease of use comes from **shared objects**, such as **Regional Edge Proxies** which utilize well-known public IP addresses. However, while this shared infrastructure enhances scalability and efficiency, it can also present risks if leveraged by attackers. 

For instance:

1. Customer(x) has tenant(x) in XC with a Load Balancer pointing to their public IP origin servers. These may be behind a perimeter firewall nat (as diagrammed below) or be actual public IPs on the servers. 
2. Customers perimeter firewall is configured to **deny all inbound traffic to public IP for site1.example.com**
3. Perimeter Firewall is configured to **allow inbound traffic to public IP for site1.example.com for XC IP's**. (which is a well known and public shared IP range) 

[XC Proxy IP's](https://docs.cloud.f5.com/docs-v2/platform/reference/network-cloud-ref)

This setup is generally considered a minimum best-practice because only traffic sourced from XC is allowed but in reality, this may not be enough from a security perspective. 

   ![XC-RE-Client](/xc-images/xc-re-client.png)


**Problem**
Another account/tenant(y) within XC could create a load balancer and point to the public IP or DNS name of the origin pools for tenant(x). The attacker must know or learn the actual origin servers IP, or network segment to perform this attack. 

In addition, what if the origin pool in tenant(x) is pointing to a DNS name that resolves to public IP's? This is common with SaaS API gateways such as AWS and Azure to name a few and these gateways all use the same DNS name for the gateway respective to their cloud. Same DNS = Same IP's = Easy to learn or guess Origin IP's. For instance a common flow where a customer is using XC for WAF/WAAP and a 3rd party SAAS solution for an APIGW, may be Client-->XC(WAAP)-->APIGW(pub-ip)-->API as shown in the diagram below. 

In this default configuration, an attacker could learn the customers public NAT IP and add it to their Origin Pool. They can now instantiate attacks from their tenant(y) which will be sourced from the XC IP's and allowed by the customer(x) perimeter firewall. 

  ![XC-RE-Attack](/xc-images/xc-re-attack.png)

**Solutions**
 
I've been thinking about giving some recommendations to our WAAP customers:
 
    1- Use server-side MTLS: If customer has the capability, we can recommend adding additional layer of source validation, a self-signed cert would add a lot of value here. (note: Until today, only F5 supports this. Other competitors only support client-side MTLS)
    2- Use private link as much as possible: There are a lot of advantages for using private link. The firewall rules can be simplified to allow traffic only from private links and private links are accessible only from the tenancy.
    3- Recommending CE option: comes with additional cost, not only for CE nodes but also for traffic.

    If customer origin has L7 aware service, add HTTP request header.  There are additional enhancement requests in for review.  However the 3 ways you outlined are also valid options.  Likely a combination of these 4 would be recommendations.