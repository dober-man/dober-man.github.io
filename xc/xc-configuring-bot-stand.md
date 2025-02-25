---
layout: default
title: Configuring Bot Standard
---

## Overview

* Bot Standard injects an interstitial (blank and transparent to the user other than a bit of latency)page and gathers JS signaling from the client. The interstitial page prevents scraping of the real page and the signaling/challenge-response guarantees a human user. JS challenges are sent to the global policy server to evaluate the response. 

AI/ML comes in to play on the backend when configuring the global policy server. AI/ML is used to evaluate the reponses and that feedback is then used by XC Security Engineers to set global policy. 

* In this example a site called catalog.lib.example.com is used. 

**Note:** Bot Standard is a paid add-on to the base package in XC.

Enable Bot Standard and edit the policy to add app endpoints Typically application paths like /logon or /cart would be protected but in this scenario we needed to use "/". 

  ![App Endpoints](/xc-images/app-end.png)

**Important:** - Bot standard does not like the "Any Method" if permitting mutiple types of HTTP. In our test, when "Any" was selected, things broke badly. 
Instead define each method that is expected. In this case, there is one for HTTP GET (Document) and one for HTTP GET (XHR). 

Typically defined application paths would be created but in this scenario the root "/" site needed protection and exceptions (Exclude Pages) are made for bypass. 

The GET (XHR) addition was necessary as a second site programmaticly (AJAX) accessed catalog.lib.example.com search field to run queries. This search field is the same field that attackers were hitting. Hotels and airlines have similar requirements...think of how sites like hotels.com and kayak.com interact with the sites to pull best fares. 

 ![XHR Endpoints](/xc-images/xhr.png)

### Exclude Pages

Two exclusions were necessary in this config. The first was for the "autosuggest feature" that is part of the search field on the page. This search field makes autosuggestions as the user enters their search queries. This same field is also used by other sites to perform searches.

Several static attempts were made at the config before relying on regex for the pattern match. We ended up with this path match:

**^\/catalog\/suggest(?:\?[^#\s]*)?(?:#.*)?$** 

This regex ensures that the string starts with /catalog/suggest and optionally includes:

* A query string (e.g., ?key=value&other=123)
* A fragment identifier (e.g., #section1)

**Example Matches**

✔ /catalog/suggest

✔ /catalog/suggest?foo=bar

✔ /catalog/suggest#heading

✔ /catalog/suggest?foo=bar#heading

**What It Does NOT Match**
✖ /catalog/suggestions (extra characters)

✖ some/catalog/suggest (missing leading /)

✖ /catalog/suggest (extra space at the end)


The second exception is for an assets directory on the server which contained site related config files, images, scripts. We ended up with this regex after a few static definition attempts. Apparently bot standard really likes regex. 

**^\/assets\/.*$**

This regex ensures that the string starts with /assets/, followed by anything.

**What Does It Match?**

✔ /assets/image.png

✔ /assets/styles.css

✔ /assets/js/script.js

✔ /assets/some/deep/path/file.txt

**What It Does NOT Match**

✖ /asset/image.png (missing s in assets)

✖ /public/assets/image.png (does not start with /assets/)

✖ assets/image.png (missing leading /)

This config stopped the bots, solved the XHR issue and allowed for ajax calls to still function. 