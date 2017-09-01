---
title: SSO environment architecture
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: introduction_sso-environment-architecture.html
summary:
---

The New School's dependence on its single sign-on environment continues to grow, making the availability of the environment ever more important. The CAS 5 server environment will be designed for high availability by ensuring that each component of the environment is sufficiently redundant to make the service resilient to multiple component failures and to enable routine maintenance on the environment to be performed without incurring service downtime. Figure 1 below highlights the principal aspects of this design.

{% include image.html file="about/fig01-devel-env.png" alt="Development Environment Diagram" caption="Figure 1. The New School CAS 5 development environment" %}

Starting with the red circle near the top of the figure, the user- (or client-) facing domain name for the single sign-on service will resolve to a virtual address on the F5 load balancers. In the development environment, this domain name will be ***casdev.newschool.edu***. The F5s are deployed in a high availability configuration between our two primary data centers, and fail over automatically in the event the primary unit goes down.

Multiple CAS servers will be deployed in an active-active configuration. A distributed ticket registry (cache) that replicates all tickets to all servers will be used to ensure that a ticket can be located from any server (the server that is asked to validate a ticket may not be the same server that originally created it). A distributed service registry that replicates all registered services to all servers will be used to ensure that all servers support the same set of services. And finally, a configuration server will be used to ensure that server configuration settings are kept in sync across all the servers. To eliminate the need to replicate Java servlet container sessions across servers (a complex and unreliable process), session affinity will be enabled on the F5s. This option tells the F5s to remember which server a new client is first directed to, and direct all requests from that client to that server for a short period of time.

The production environment will require a minimum of four CAS servers, two in each primary data center, to ensure that redundancy is maintained even if one data center goes offline. However, as shown in the figure, the development environment (as well as the test environment) can get by with just three servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03.newschool.edu***), which conserves VMware resources while still enabling us to validate replication operations in the more complex "N > 2" case.

Two user directories are used to provide authentication services: Active Directory and Luminis LDAP. The Active Directory environment is already configured for high availability, with two domain controllers in each data center (and a fifth one in Paris) fronted by a virtual address (***zuul.newschool.edu***) on the F5s. The Luminis LDAP environment is not currently configured for high availability; there is only a single server instance. However, the instance is located behind a virtual address (***janus.newschool.edu***) on the F5s, and the underlying technology (OpenDJ) supports replication, so enabling high availability should be relatively straight forward (doing so is outside the scope of this document, however).

To facilitate development and testing, the development environment will also include two single sign-on enabled applications. Shown at the top left of Figure 1 as blue boxes, ***casdev-casapp.newschool.edu*** will be a CAS-enabled Apache web server, and ***casdev-samlsp.newschool.edu*** will be a SAML2-enabled Apache web server.

## References

* [CAS 5: High Availability Guide (HA/Clustering)][casdoc-ha-guide]

{% include reflinks.md %}
