---
layout: default
title: Configuring Service Policies for Geolocation and IP
---

## Overview

Service Policies are an incredibly powerful matching mechinism for both client and server side traffic. They can operate at layer 3-7 and are designed from the point of view of the proxy. Service policy can be designed for a single proxy or a set of proxies or all proxies matching a given label expression.

[XC Security Overview] https://docs.cloud.f5.com/docs-v2/platform/concepts/security

## Goal

* Allow only defined Geolocations and IP's to Load Balancer
* Disallow all others 

### Defining the Allowed Source IPs and Networks

1.  Starting with a functional HTTP or TCP Load Balancer, navigate to: 

    **Multi-Cloud App Connect > Security > Service Policies > Service Policies > "Add Service Policy"** 

2. Give the Service Policy a meaningful **Name** and make a "Server Selection". For this example choose **Any Server**. 

    > **Note:** "Server" is a backend term in this context. 

3. Under the **Rules > Select Policy Rules**  section, change the default dropdown menu item from "Custom Rule List" to **"Allowed Sources"**.

    ![Service Policy](/xc-images/ip-geo.png)

4. Under "IPv4 Prefix List" click the blue **Configure**. 
    * Add your various allowed source IP's and Networks here and click "Apply". 

5. Scroll down to **Country List** and start typing "United States". Choose **"United States"**.


    ![Policy Rule](/xc-images/rule.png)


6. Click "Apply" and then "Save and Exit". 

    > **Important:** When you create an "Allow" type Service Policy there is an inherent default deny. You DO NOT have to configure a second Service policy for the Deny. 


7. By default you should now see the following policies


   ![Default Service Policies](/xc-images/sp.png)


8. Navigate to the Load Balancer that you want to apply the service policy to.

   **Multi-Cloud App Connect > Manage > Load Balancers**

9.  Under "Actions" choose "Manage Configuration" > "Edit Configuration" and scroll down to the **"Common Security Controls"** section. Click "Edit" and choose your new service policy. 
   ![Security Controls](/xc-images/sec-con.png)
10. Click **"Save and Exit"**. 


### Testing the Allowed Source IPs and Networks

1. Make sure the site works from the IP addresses defined in the Service Policy. The way this service policy is written, it includes **Any** USA IP address plus an additional network and source IP that we added for an example. 

    > **Note:** In practice, if the IP and subnet ranges defined in the rule were actually from the USA, they would match the geo allow and not need additional definition. They are only provided as example of how to add an IP and subnet.  

2. Test from a client that is outside the US. I used a VPN to do this and connected from a London IP. The service policy default deny was triggered. 

   ![Request Log](/xc-images/rl.png)


## Applying the Service Policy to all LB's in the Namespace

By default when a new LB is created the setting for "Service Policies" is **"Apply Namespace Service Policies"**. This makes setting a Namespace Service Policy straightforward. 

   ![Default SP](/xc-images/default-sp.png)

1. Navigate to Service Policies > **Active Service Policies** and click **"Select Active Service Policies"** add the desired service policy. 

   ![Active SP](/xc-images/active-sp.png)

2. When Applying Service Policy to the Namespace, make sure to change the **Default Action** to **Deny**.  

   ![Ending](/xc-images/ending.png)

Edit service policy and change default ending to Deny. 
    > **Note:** Changing the ending to default deny is only necessary when applying Namespace Service Policies. When applying a "custom" Service policy directly to a load blanancer there is an inherent default deny. 
3. Ensure all load balancers are configured to  **Apply Namespace Service Policies"**.



