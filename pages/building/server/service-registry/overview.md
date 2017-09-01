---
title: Adding a service registry
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_service-registry_overview.html
summary: A service registry must be added to the server so that client services can be declared and configured.
---

The CAS server includes a service management facility that allows CAS server administrators to declare and configure which services (CAS clients) may use the server, and how they may use it. The core component of the service management facility is the service registry that stores information about registered services including how the services must authenticate users, which users may access the service and under what conditions, data about authorized users the services may access, and so on.

The basic CAS server built in the previous section does not include a service registry (there is a line in `cas.properties` to enable a built-in registry, but it is commented out). Before we can build and use any test clients, it's necessary to add a service registry to the server. For our initial testing, we will add a simple registry that uses JSON files to describe services; we will replace this with a more robust registry when we configure the servers for [high availability][high-avail_overview].

## References

* [CAS 5: Service Management][casdoc-svc-mgmt]

{% include reflinks.md %}
{% include links.html %}
