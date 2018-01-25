---
title: High availability
last_updated: December 21, 2017
sidebar: main_sidebar
permalink: high-avail_overview.html
summary: Now that everything is working correctly in a single-server environment, the steps to enable multiple servers to operate in a pooled configuration can be performed.
---

As explained in the [introduction][introduction_sso-environment-architecture], one of the implementation goals for this environment is to have "high availability (fault-tolerant) everything." To that end, the environment has been built with a pool of servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) behind a load balancer. To enable these servers to work together in an active-active configuration, where any server in the pool is capable of servicing any request and the pool can continue to service requests even if one or more servers is unavailable, the following tasks must be performed:

1. A distributed ticket registry (cache) that replicates all tickets to all servers must be created to ensure that a ticket can be located from any server (the server that is asked to validate a ticket may not be the same server that originally created it).
2. A distributed service registry that replicates all registered services to all servers must be created to ensure that all servers support the same set of services.
3. Distributed storage for CAS SAML IdP metadata must be created to ensure that all the servers behave the same way, and distributed storage for cached SAML SP metadata must be created to ensure that all servers know about all SPs.
4. A distributed configuration property storage solution that replicates all configuration settings to all servers must be created to ensure that all servers are configured the same way.

CAS 5 supports a variety of caches, databases, and configuration servers to implement ticket registries, service registries, and configuration property storage. For our implementation, we will use the [MongoDB][mongodb-org] NoSQL database, which offers a lightweight implementation with built-in replication and fault tolerance features that can be installed on the same virtual machines that we're using to run the CAS servers.

## References

* [CAS 5: Ticketing][casdoc-ticketing]
* [CAS 5: Service Management][casdoc-svc-mgmt]
* [CAS 5: Configuration Server][casdoc-config-server]

{% include reflinks.md %}
{% include links.html %}
