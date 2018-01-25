---
title: Setting up the service registry
last_updated: December 20, 2017
sidebar: main_sidebar
permalink: high-avail_service-registry_overview.html
summary: A distributed service registry, accessible to all CAS servers, will be used to ensure that every server has the most up-to-date information about authorized services, and to allow the registry to be maintained from a single administration point.
---

The [JSON service registry][building_server_service-registry_overview] works well in a single server environment. But in an environment with a pool of servers, it doesn't. Most significantly, the [service management webapp][building_svcmgmt_overview] won't work correctly, because any modifications it makes to the service registry will only take effect on the particular pool server where the webapp session is running. The other servers' registries will be out of date until some out-of-band process (manual or automated) can update them with the new information. That update process however is complicated by the fact that the service management webapp runs on every CAS server (for high availability/fault tolerance), so different changes can be made on different servers, requiring the synchronization process to be capable of performing N-way merges and resolving any resultant conflicts.

To solve this problem, we will replace the JSON service registry with one stored in MongoDB. Each CAS server in the pool will load (and reload) its list of authorized services from the database and every instance of the service management webapp will write to the database, thus ensuring that all servers in the pool always have up-to-date, identical information about authorized services.

{% include reflinks.md %}
{% include links.html %}
