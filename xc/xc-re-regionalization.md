---
layout: default
title: Configuring Regionalization for XC RE's
---

## Overview

By default, an XC load balancer deployed to the "Internet" (also known as Regional Edge Deployment) is replicated across all Regional Edges within the F5 ADN. Certificates and decryption occurs at each Regional Edge participating in a load balancer deployment. This broad deployment helps absorb DDoS attacks and mitigate regionalized issues but may conflict with compliance mandates such as HIPAA or FedRAMP if traffic is being decrypted outside of a given territorial boundary.

Fortunately, an XC load balancer can be regionalized which means the the certificates required for decrypt and decryption itself will only occur on selected Regional Edges grouped into a Virtual Site. The other RE's not defined act simply as ingress routers to our App Delivery Network (ADN) and will route traffic to the regionalized RE's. This customization supports compliance with regional data residency and privacy regulations since the certificates and decryption occurs within a custom-defined territorial boundary.

[XC Regionalization](https://community.f5.com/kb/technicalarticles/f5-distributed-cloud---regional-decryption-with-virtual-sites/307273)