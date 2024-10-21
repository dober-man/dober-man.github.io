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


**Risk**

Another account/tenant(y) within XC could create a load balancer and point to the public IP or DNS name of the origin pools for tenant(x). The attacker must know or learn the actual origin servers IP, or network segment to perform this attack. 

In addition, what if the origin pool in tenant(x) is pointing to a DNS name that resolves to public IP's? This is common with SaaS API gateways such as AWS and Azure to name a few and these gateways all use the same DNS name for the gateway respective to their cloud. Same DNS = Same IP's = Easy to learn or guess Origin IP's. For instance a common flow where a customer is using XC for WAF/WAAP and a 3rd party SAAS solution for an APIGW, may be Client-->XC(LB-WAAP)-->APIGW(pub-ip)-->API. 

In this default configuration, an attacker could learn the customers public NAT IP and add it to their Origin Pool. They can now instantiate attacks from their tenant(y) which will be sourced from the XC IP's and allowed by the customer(x) perimeter firewall. 

  ![XC-RE-Attack](/xc-images/xc-re-attack.png)

**Mitigation**
 
There are at least 4 ways to mitigate this risk. 

 1. **L7 Header** - If the origin servers (on-prem or SAAS) have something in front of them that is "L7 aware" or they themselves can be configured to do header valiudation, a custom HTTP request header could be injected into the flow by the load balancer in "tenant x". Tenant y would not know or be able to see this header. Of course traffic not containing this header would still make it all the way to the L7 aware service before being dropped. While this would suffice for a L7 DoS or or other L7 type attack, it would not help with a L3/4 type attack which could still make it's way through the infrastructure.  
 
   ![Custom Header](/xc-images/header.png)
 

2. **MTLS** - A unique differentiator for F5 XC is our ability to use server-side MTLS. If customer has the capability on the Web Server/Service or something in front of it similar to the previous L7 header example, then we can add an additional layer of source validation by using mutual certificate authentication. Even a self-signed cert would add a lot of value here. No cert = no layer 7 access to the app or service. This does not prevent an L3/4 attack but will prevent unwanted application access. 

   ![mtls](/xc-images/mtls.png)

3. **Customer Edge** (CE) proxies are deployable software that creates a private mesh back to our Application Delivery Network (ADN). These come with additional cost and need to be deployed at each location, thus creating a private mesh or overlay network that is unavailable outside of the tenant. in this scenario, the attacker traffic could potentially make it to the public IP of (or in front of) the CE and be dropped, thus protecting the application itself but still potentially allowing bad L3/4. 


   ![ce](/xc-images/ce.png)


4. **Private Link** is a paid for addon to XC and can be utilized for connectivity between XC, clients and resources.  This has alot a lot of advantages when dealing with regulatory or other security compliances. 
The perimeter firewall rules can be simplified to allow traffic only from Private Links and Private Links are accessible only from the tenancy. Private Links can solve for L3-L7 attacks in that the link is entirely private. 

[XC Private Link](https://www.f5.com/pdf/solution-profiles/introducing-f5-distributed-cloud-private-link-solution-overview.pdf)

   ![Private Link](/xc-images/private-l.png)

  
> **A word on L3/4 DDoS:** L3/4 attacks were brought up several times above when talking about the technicalities of each mitigation method. While a L3/4 attack is not always distributed by nature, most are. One very important concept to keep in mind is the fact that XC natively provides L3/4 DDoS mitigation at our Regional Edges. Even in the examples above where "attack" traffic could make it all the way to the app or at least to the perimeter, if it was a true DDoS, this would get picked up by our Regional Edges and automatically mitigated. 

   ![DDoS](/xc-images/ddos.png)

   