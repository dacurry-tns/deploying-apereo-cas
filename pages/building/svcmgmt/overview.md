---
title: Building the management webapp
last_updated: October 11, 2018
sidebar: main_sidebar
permalink: building_svcmgmt_overview.html
summary: Build and install the separate management webapp to make it easier to manage the service registry and prepare for managing the service registry in a high availability environment.
---

The CAS management webapp is a web-based GUI that allows CAS administrators to create, modify, and delete service definitions in the service registry. It is implemented as a completely separate web application built independently of the CAS server. It can be deployed in the same Java Servlet container that is running the CAS server, or it can be deployed in a completely separate container. Either way, the operation of the CAS server does not depend in any way on the state of the management webapp.

The management webapp isn't strictly needed when using a file-based service registry such as the JSON registry we have been using to this point, although it will help avoid JSON syntax errors and similar mistakes. Where the webapp becomes important is when using storage back ends such as databases for the service registry. In that configuration, the webapp becomes the interface fronting the CRUD operations that deal with the back end storage system.

Since [implementing high availability][high-avail_overview] will require replacing the JSON file-based service registry with something else that is suited for use in a multiple server, clustered environment, the webapp is going to be a key component of our environment.

## References

* [CAS 5: Services Management Webapp][casdoc-svc-mgmt-webapp]

{% include reflinks.md %}
{% include links.html %}
