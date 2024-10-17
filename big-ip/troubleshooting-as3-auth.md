---
layout: default
title: Troubleshooting AS3 Auth
---

### Troubleshooting OpenShift and BIG-IP AS3 Authentication Failure

A 401 Unauthorized error is typically due to an authentication configuration. Follow these steps to verify AS3 credentials are valid. 

#### 1. Check Service Account Permissions
Ensure the service account you set up has the necessary permissions on the BIG-IP to access AS3. Specifically, check that the user associated with the service account has the `admin` role or any role with access to `/mgmt/shared/appsvcs/info`.

#### 2. Verify Authentication Details
Confirm that the credentials (username and password) used by the service account are correct and properly configured in your Helm deployment values or Kubernetes secrets. Any misconfiguration could lead to the `401 Unauthorized` error.

#### 3. Ensure API Endpoints Are Accessible
Test the connectivity to the AS3 API manually using `curl` or Postman. For example, you can run:

```bash
curl -k -u <username>:<password> https://<BIGIP_IP>/mgmt/shared/appsvcs/info
```

This will help verify whether the credentials are correct and whether the AS3 API is reachable.

#### 4. Check BIG-IP User Roles
Verify that the user assigned to your OpenShift service account has the necessary role to perform AS3 operations. You can inspect user roles under System > Users > Users List in the BIG-IP GUI and assign a higher role if necessary.

#### 5. Check Helm Chart Values

Review the values you passed to the Helm deployment to ensure that they are correct, especially the BIG-IP authentication details and the CIS (Container Ingress Services) configuration for AS3.

#### 6. Log into the BIG-IP and Check Logs

Investigate the BIG-IP system logs for more detailed information on the 401 error. Logs related to AS3 can be found in:

    /var/log/restjavad.0.log
    /var/log/restnoded/restnoded.log

#### 7. Check BIG-IP Rest API Setup:

Make sure that the AS3 is correctly installed and functioning on the BIG-IP device. You can verify this by accessing:

https://<BIGIP_IP>/mgmt/shared/appsvcs/info

This endpoint should return AS3-related information if AS3 is working properly.

#### 8. BIG-IP AS3 Version Compatibility:

Ensure that AS3 version 3.53.0 is compatible with the version of F5 BIG-IP you’re using. Sometimes, incompatibilities between AS3 versions and the BIG-IP version can cause authentication failures.

#### 9. Check for Possible IP Restrictions:

Verify if there are any IP restrictions or firewall rules between OpenShift and the BIG-IP devices that might be blocking or interfering with communication.

#### 10. CIS Configuration:

Ensure the Helm chart and F5 Container Ingress Services (CIS) configurations (like the bigip-username and bigip-password) are correctly set in your deployment. Incorrect configurations here could lead to failed authentication.