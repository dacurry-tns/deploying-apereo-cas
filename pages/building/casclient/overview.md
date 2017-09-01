---
title: Building the CAS client
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_casclient_overview.html
summary: To facilitate development and testing, a client application that interacts with the CAS server is needed.
---

Now that a basic CAS server is up and running, we can build a client application to talk to it.

Our CAS client will be an Apache HTTPD web server that offers both public content that anyone can access, and "secure" content that can only be accessed by authenticated and authorized users. The CAS server will be used to perform those authentication and authorization decisions.
