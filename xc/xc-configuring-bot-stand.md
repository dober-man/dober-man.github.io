---
layout: default
title: Configuring Bot Standard
---

## Overview

* Bot Standard injects an interstitial page and gathers JS signaling from the client. The interstitial prevents scraping and the signaling gurantees a human user. JS challenges are sent to the global policy server to evaluate the response. AI/ML comes in to play when configuring the global policy server. AI/ML is used to evaluate the reponses and that feedback is then used to set global policy. 

* In this example a site called catalog.lib.example.com is used. 

**Note:** Bot Standard is a paid add-on to the base package in XC

1.  Enable bot standard and edit the policy to add app endpoints. 

  ![App Endpoints](/xc-images/app-end.png)

**Important:** - Bot standard does not like the "Any Method" if permitting mutiple types. When we selected "Any" things broke badly. 
Instead define each method that is expected. In this case, there is one for HTTP GET (Document) and one for HTTP GET (XHR). 

Typically defined paths would be created but in this scenario the root "/" site needed protection and exceptions (Exclude Pages) are made for bypass. 

THe GET (XHR) addition was necessary as a second site programmaticly (AJAX) accessed catalog.lib.example.com search field to run queries. THis search field is the same field that attackers were hitting. 

 ![XHR Endpoints](/xc-images/xhr.png)

### Exclude Pages
Two exclusions were necessary. The first was for the "autosuggest feature" that is part of the search field on the page.

1. Several static attempts were made at the config before relying on regex for the pattern match. We ended up with this path match:

**^\/catalog\/suggest(?:\?[^#\s]*)?(?:#.*)?$** 

This regex ensures that the string starts with /catalog/suggest and optionally includes:

* A query string (e.g., ?key=value&other=123)
* A fragment identifier (e.g., #section1)

Example Matches

✔ /catalog/suggest

✔ /catalog/suggest?foo=bar

✔ /catalog/suggest#heading

✔ /catalog/suggest?foo=bar#heading

What It Does NOT Match
✖ /catalog/suggestions (extra characters)
✖ some/catalog/suggest (missing leading /)
✖ /catalog/suggest (extra space at the end)

2. The second exception is for an assets directory on the server which contained site related config files, images, scripts. 
We ended up with this regex after a few static definition attempts. Apparently bot standard really likes regex. 

**^\/assets\/.*$**

What Does It Match?

This regex ensures that the string starts with /assets/, followed by anything.

✔ /assets/image.png
✔ /assets/styles.css
✔ /assets/js/script.js
✔ /assets/some/deep/path/file.txt

What It Does NOT Match
✖ /asset/image.png (missing s in assets)
✖ /public/assets/image.png (does not start with /assets/)
✖ assets/image.png (missing leading /)

This config solved the issue and allowed for ajax calls to still function. 