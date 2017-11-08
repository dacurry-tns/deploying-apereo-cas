---
title: Update the service registry
last_updated: November 8, 2017
sidebar: main_sidebar
permalink: building_server_dashboard_update-the-service-registry.html
summary:
---

Since we are using the CAS server to protect the endpoints, we need to create a service definition for the dashboard.

## Create a service definition for the dashboard

Create the file `etc/cas/services/CASAdminDashboard-1509646291.json` (replace `1509646291` with the current `date +%s` or `YYYYMMDDhhmmss` value) in the `cas-overlay-template` directory on the master build server (***casdev-master***) with the following contents:

```json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^https://casdev.newschool.edu/cas/status/dashboard(\\z|/.*)",
  "name" : "CAS Admin Dashboard",
  "id" : 1509646291,
  "description" : "CAS dashboard and administrative endpoints",
  "evaluationOrder" : 5000
}
```

There is no need to create service definitions for all the other endpoints underneath `/status`; they will all be authenticated by this one. Note, however, that any attempt to access those other endpoints before accessing the `/status/dashboard` endpoint and authenticating to the CAS server will fail.

## References

* [CAS 5: JSON Service Registry][casdoc-svc-reg-json]

{% include reflinks.md %}
