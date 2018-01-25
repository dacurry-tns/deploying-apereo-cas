---
title: Setting up the ticket registry
last_updated: December 18, 2017
sidebar: main_sidebar
permalink: high-avail_ticket-registry_overview.html
summary: A distributed ticket registry, accessible to all CAS servers in the environment, will be used to ensure that tickets can be located (and validated) by any server in the environment, not just the server that created it.
---

To support high availability/fault tolerance, the CAS server environment has been built with a pool of servers behind a load balancer. However, as configured up to this point, each server in the pool behaves as an independent entity. This is immediately obvious when trying to access the "secure" content on either ***casdev-casapp*** or ***casdev-samlsp*** when more than one of the servers in the pool is up and running:

1. When a user attempts to access a CAS-protected application (or CAS-protected content within the application), the application checks to see if the user has provided a CAS Service Ticket (ST) as authorization. If an ST has not been provided (as happens when the application is first accessed), the application sends the user (usually with a web browser redirect) to the CAS server to obtain one.
2. When the user accesses the CAS server, the load balancer (which holds the IP address registered to the CAS server's public DNS host record) will connect the user to one of the serves in the pool using whatever load balancing strategy has been configured (e.g., round-robin).
3. The CAS server will query the user's client (usually by looking for a web browser cookie) to see if the user has a CAS Ticket Granting Ticket (TGT):
    1. If the user does not have a TGT, then CAS will prompt the user to enter his or her credentials (username, password, multi-factor authentication, etc.) and, upon successful authentication, create a TGT for the user.
    2. If the user already has a TGT, then CAS will attempt to validate it (make sure it can be decrypted, hasn't expired, etc.) and, if validation is successful, allow the user to proceed without having to enter his or her credentials again. **Problem:** By default, the CAS server stores the information it needs to validate TGTs (the *ticket registry*) in memory. Thus, if one server in the pool created a TGT, another server in the pool will be unable to validate it, because it doesn't have access to that information.
4. Once the TGT has been created/validated, the CAS server uses it to create an ST for the application, and sends the user (again, usually with a web browser redirect) back to the application.
5. The application takes the ST provided by the user and sends it via a back channel (user-transparent) communication to the CAS server to be validated. The CAS server will use the TGT and other information to validate the ST, and return the results to the application. This is also the point at which any user attributes (e.g., from Active Directory or LDAP) are returned to the application. **Problem:** The CAS server needs information from the ticket registry to validate STs. Since, as mentioned above, this information is stored in memory by default, CAS servers cannot validate STs created by other servers in the pool because they do not have access to the necessary information.
6. Once the ST has been validated, the application allows the user to access the protected content.

The problems identified in steps 3(b) and 5 above are why, when testing the CAS client and SAML client in previous sections, we always shut down all but one of the CAS servers in the pool. (For more details on the steps above, including flow diagrams, see [CAS Protocol][casdoc-cas-protocol].)

To solve the problem of each server in the pool not having the information needed to validate TGTs and STs created by other servers, we will replace the default in-memory ticket registry with one stored in MongoDB. Each CAS server in the pool will save its tickets to the database, and any other server in the pool will be able to access the tickets created by other servers when necessary.

{% include reflinks.md %}
{% include links.html %}
