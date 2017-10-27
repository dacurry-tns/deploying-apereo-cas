---
title: Building the SAML client
last_updated: October 25, 2017
sidebar: main_sidebar
permalink: building_samlclient_overview.html
summary: To facilitate development and testing, a client application that interacts with the CAS server via SAML is needed.
---

Now that SAML2 Identity Provider (IdP) support has been [added to the CAS server][building_server_saml_overview], we can build a Service Provider (SP) client application to talk to it.

Our SP will be an Apache HTTPD web server that offers both public content that anyone can access, and "secure" content that can only be accessed by authenticated and authorized users. The IdP functionality of the CAS server will be used to perform those authentication and authorization decisions.

{% include links.html %}
