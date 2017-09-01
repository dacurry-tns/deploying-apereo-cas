---
title: Install HTTPD and PHP on the client servers
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_httpd-php_overview.html
summary: A CAS-enabled Apache HTTPD server and a SAML2-enabled Apache HTTPD server will be used as clients to test the CAS server. PHP will be used to help examine user attribute data passed back to the clients from the server.
---

To test the operation of our CAS server environment, we need a client application that will prompt for a user name and password (and perhaps other authentication factors) and then contact the CAS server to authenticate that user, and request/accept attribute information describing the user. Since the development environment will support both the CAS and SAML2 protocols for this purpose, we need two client applications&mdash;one for each protocol.

There are a variety of open-source and commercial libraries and toolkits that can be used to build CAS or SAML2 support into client applications, but building client applications from scratch is beyond the scope of this project. Therefore, we will instead use the Apache HTTPD server as our client, with CAS support enabled by the Apereo `mod_auth_cas` plug-in, and SAML2 support enabled by the Shibboleth Consortium's Service Provider distribution. We will install the clients on two separate servers, ***casdev-casapp.newschool.edu*** and ***casdev-samlsp.newschool.edu***.

This section describes the Apache HTTPD and PHP (for processing attributes) installation steps common to both servers; the protocol-specific configuration of each server will be described in later sections.

## References

* [Apereo mod_auth_cas][mod_auth_cas]
* [Shibboleth Service Provider][shibboleth-sp]
* [CAS 5: CAS Clients][casdoc-cas-clients]
* [Wikipedia: Libraries and toolkits to develop SAML actors and SAML-eanbled services][wikipedia-saml-libs]

{% include reflinks.md %}
