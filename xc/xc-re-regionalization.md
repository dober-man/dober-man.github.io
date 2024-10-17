---
layout: default
title: Configuring REgionalization for XC RE's
---

## Overview

By default, a load balancer deployed to the "Internet" (also known as Regional Edge Deployment) is replicated across all Regional Edges within the F5 ADN. This broad deployment helps absorb DDoS attacks and mitigate regionalized issues but may conflict with compliance mandates such as HIPAA or FedRAMP.

Fortunately, an XC load balancer can be customized to replicate only to selected Regional Edges where the other RE's not defined act simply as ingress routers to our App Delivery Network (ADN). This customization supports compliance with regional data residency and privacy regulations since the certificates and decryption occurs within a territorial boundary.

[XC Regionalization](https://community.f5.com/kb/technicalarticles/f5-distributed-cloud---regional-decryption-with-virtual-sites/307273)