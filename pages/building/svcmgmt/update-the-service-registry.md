---
title: Update the service registry
last_updated: November 8, 2017
sidebar: main_sidebar
permalink: building_svcmgmt_update-the-service-registry.html
summary:
---

Since we are using the CAS server to protect the services management webapp, we need to create a service definition for it.

## Create a service definition for the webapp

Create the file `etc/cas/services/CASServiceManagement-1510002272.json` (replace `1510002272` with the current `date +%s` or `YYYYMMDDhhmmss` value) in the `cas-overlay-template` directory on the master build server (***casdev-master***) with the following contents:

```json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^https://casdev.newschool.edu/cas-management(\\z|/.*)",
  "name" : "CAS Services Management",
  "id" : 1510002272,
  "description" : "CAS services management webapp",
  "evaluationOrder" : 5500
}
```

## References

* [CAS 5: JSON Service Registry][casdoc-svc-reg-json]

{% include reflinks.md %}
