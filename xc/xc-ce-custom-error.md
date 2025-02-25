---
layout: default
title: Configuring a Custom Error Page on CE
---

## Overview

* By default the CE (site or virtual site) LB does not respond with a default error page for Service Policy denies. 
* LB Config setting "Disable Default Error Pages" only applies to RE based LB.

### Default Conditions: 

* LB is advertised on a CE
* In this example the Service Policy is set to permit allowed-IP's to allowed-URI's. 
* Disable Default Error Pages is not checked
* Service Policy Deny produces a blank page to the client browser
* Requests can be seen under the logs in XC Console

"Disable default error pages" is not selected by default when creating an LB with Custom Error Response Options. In that state the error page from a "Service Policy Deny" displays as a blank page to the client browser. This only occurs if the LB is advertised on CE.

### Default Conditions: 

You can only configure custom response pages for errors from the web server (403, 503 etc)



Custom Error Page

```
<html><head><title>Error Page</title></head>
<body>The requested URL was rejected. Please consult with your administrator.<br/><br/>
Your support ID is {{request_id}}<h2>Error 403 - Forbidden<br/><br/><a href='javascript:history.back();'>[Go Back]</a></body></html>
```
