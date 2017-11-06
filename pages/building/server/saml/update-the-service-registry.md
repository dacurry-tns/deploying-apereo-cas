---
title: Update the service registry
last_updated: November 6, 2017
sidebar: main_sidebar
permalink: building_server_saml_update-the-service-registry.html
summary:
---

When the CAS server starts and initializes the SAML IdP, it creates a new (undocumented, as of this writing) endpoint, `${cas.server.prefix}/idp/profile/SAML2/Callback`. It then checks the service registry to see if there is an existing service definition whose `serviceId` will match then endpoint and allow access. If there isn't one, the CAS server will create a new service definition for the endpoint, and save it to the service registry. If the CAS server does not have permission to create new entries in the service registry for whatever reason, then the save will fail, and the server will not start. The error messages in the CAS log file will be somewhat cryptic in this case, because they won't refer to SAML or the IdP at all. But the relevant lines will look something like this:

```
=============================================================
WHO: audit:unknown
WHAT: IO error opening file stream.
ACTION: SAVE_SERVICE_FAILED
APPLICATION: CAS
WHEN: Ddd Mon DD hh:mm:ss zzz YYYY
CLIENT IP ADDRESS: unknown
SERVER IP ADDRESS: unknown
=============================================================

(...lots of stack trace output...)

Caused by: java.io.FileNotFoundException: /etc/cas/services/RegexRegisteredService-6805904835673174978.json (Permission denied)
```  

As luck would have it, back when we [first set up our service registry][building_server_service-registry_overview], we created a "wildcard" service definition (`HTTPSandIMAPSwildcard-1503925297.json`) that will match the endpoint above, so the CAS server will not create a new service definition and try to save it in the registry. However, while a wildcard service definition is fine in a development or test environment, we won't have such a thing in our production environment.

## Create a service definition for the IdP endpoint

To avoid any risk of the server failing to start as described above, and also to make it clear that the IdP endpoint is a "desired" service, we will explicitly create a service definition file for it.

Make a copy of `etc/cas/services/HTTPSandIMAPSwildcard-1503925297.json` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and call it `SAML2CallbackProfile-1509029745.json` (replace `1509029745` with the current `date +%s` or `YYYYMMDDhhmmss` value):

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# cp -p etc/cas/services/HTTPSandIMAPSwildcard-1503925297.json etc/cas/services/SAML2CallbackProfile-1509029745.json
```

Then edit `etc/cas/services/SAML2CallbackProfile-1509029745.json` and do the following:

1. Change the `serviceId` property to `https://casdev.newschool.edu/cas/idp/profile/SAML2/Callback.+` (note the `.+` regular expression component on the end).
2. Change the `name` property to `SAML Authentication Request`.
3. Change the `id` property to a unique value (make sure this value matches the one in the filename).
4. Change the `evaluationOrder` property to a value smaller than the other services' values, to (hopefully) ensure that this definition will always match the endpoint.
5. Add a comment to explain what the definition is for (optional).

When done, the file should look something like this:

```json
{
  /*
   * The CAS SAML IdP creates this endpoint as part of its initialization
   * process at server startup time. If the service registry doesn't already
   * contain an entry whose serviceId matches the endpoint, CAS will create
   * a new service definition and save it to the registry. If the CAS server
   * doesn't have write access to the registry, then the save will fail and
   * the server will not start.
   *
   * To avoid that situation, and to make it clear that this endpoint is a
   * "desired" service, it is defined explicitly here.
   */
  "@class" :            "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" :         "https://casdev.newschool.edu/cas/idp/profile/SAML2/Callback.+",
  "name" :              "SAML Authentication Request",
  "id" :                1509029745,
  "evaluationOrder" :   100
}
```

{% include links.html %}
