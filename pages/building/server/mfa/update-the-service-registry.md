---
title: Update the service registry
last_updated: September 26, 2017
sidebar: main_sidebar
permalink: building_server_mfa_update-the-service-registry.html
summary:
---

Although it's possible to enable MFA across the board for all services by setting properties in `cas.properties` (see [CAS 5: Configuration Properties: Multifactor Authentication][casdoc-mfa-props]), it's usually preferable to configure it on a per-service basis in the service registry.

## Create a second service definition for the CAS client

Make a copy of `etc/cas/services/ApacheSecuredByCAS-201700830155400.json` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and call it `ApacheSecuredByCASandDuo-201700831132700.json` (replace `201700831132700` with the current `YYYYMMDDhhmmss` value):

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# cp -p etc/cas/services/ApacheSecuredByCAS-201700830155400.json etc/cas/services/ApacheSecuredByCASandDuo-201700831132700.json
```

Then edit `etc/cas/services/ApacheSecuredByCASandDuo-201700831132700.json` and do the following:

1. Change the `serviceId` property to reflect the path to the secure area [created in the previous step][building_server_mfa_update-the-cas-client-config].
2. Change the `id` property to a unique value (make sure this value matches the one in the filename).
3. Change the `description` property to include the Duo MFA requirement.
4. Add the `multifactorPolicy` property as shown below.
5. Change the `evaluationOrder` property to a different value.

When done, the file should look something like this:

```json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^https://casdev-casapp.newschool.edu/secured-by-cas-duo(\\z|/.*)",
  "name" : "Apache Secured By CAS and Duo",
  "id" : 201700831132700,
  "description" : "CAS development Apache mod_auth_cas server with username/password and Duo MFA protection",
  "attributeReleasePolicy" : {
    "@class" : "org.apereo.cas.services.ReturnAllAttributeReleasePolicy"
  },
  "multifactorPolicy" : {
    "@class" : "org.apereo.cas.services.DefaultRegisteredServiceMultifactorPolicy",
    "multifactorAuthenticationProviders" : [ "java.util.LinkedHashSet", [ "mfa-duo" ] ]
  },
  "evaluationOrder" : 1200
}
```

The `multifactorPolicy` added here defines a single MFA provider, `mfa-duo`. It does not allow the MFA requirement to be bypassed (meaning that users not registered with Duo will not be able to log in), and it will fail "closed," meaning that if for some reason the Duo service is unavailable, users will not be able to log in.

## References

* [CAS 5: Duo Security Authentication][casdoc-mfa-duo]
* [CAS 5: Configuration Properties: Multifactor Authentication][casdoc-mfa-props]

{% include reflinks.md %}
{% include links.html %}
