---
title: Configure the service registry
last_updated: October 20, 2017
sidebar: main_sidebar
permalink: building_server_service-registry_configure-the-service-registry.html
summary:
---

Configuring the service registry requires defining the registry location in `cas.properties` and then creating service definition files for each service.

## Define the service registry in `cas.properties`

Edit the file `etc/cas/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and locate the commented-out definition of the service registry location (around line 7):

```properties
# cas.serviceRegistry.json.location: classpath:/services
```

Uncomment the line and change the property's value to `file:/etc/cas/services`:

```properties
cas.serviceRegistry.json.location:    file:/etc/cas/services
```

## Create the service registry directory

Create the directory `etc/cas/services` in the `cas-overlay-template` directory on the master build server.

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# mkdir etc/cas/services
```

## Create a service definition file

For simplicity (and to avoid worrying about the details of the service registry for the moment), create a "wildcard" service definition that will allow any HTTPS- or IMAPS-based service to make use of the CAS server. Create a file in the `etc/cas/services` directory on the master build server with the following contents:

```json
{
  /*
   * Wildcard service definition that applies to any https or imaps url.
   * Do not use this definition in a production environment.
   */
  "@class" :            "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" :         "^(https|imaps)://.*",
  "name" :              "HTTPS and IMAPS wildcard",
  "id" :                20170828090137,
  "evaluationOrder" :   99999
}
```

The CAS documentation recommends the following naming convention for JSON service definition files:

```json
JSON filename = serviceName + "-" + serviceNumericId + ".json"
```

Therefore, the filename for the wildcard service definition above should be `HTTPSandIMAPSwildcard-20170828090137.json`.

The CAS server uses [Human JSON][human-json] (Hjson), which relaxes JSON's strict syntax rules and also allows for the use of comments, to make it easier to write JSON service definitions by hand. (Later, we will build the [service management webapp][building_svcmgmt_overview] to maintain these files for us). The use of Hjson format for writing service definitions is optional; traditional JSON syntax is also supported.

The complete list of service definition properties is provided in the *Service Management* chapter of the CAS documentation, but the "interesting" fields in the definition above are:

<table>
    <colgroup>
        <col width="25%" />
        <col width="75%" />
    </colgroup>
    <tbody>
        <tr>
            <td markdown="span">`serviceId`</td>
            <td markdown="span">A regular expression describing the URL(s) where a service or services are located. Care should be taken to avoid patterns that match more than just the desired URL(s), as this can create security vulnerabilities.</td>
        </tr>
        <tr>
            <td markdown="span">`name`</td>
            <td markdown="span">A name for the service. Note that because the service definition filename is created based on this name (see above), the value of this field should never contain [characters that are not allowed in filenames][invalid-filename-chars].</td>
        </tr>
        <tr>
            <td markdown="span">`id`</td>
            <td markdown="span">Unique numeric identifier for the service definition. An easy way to ensure that these identifiers are unique is to use the date and time the service definition was created, in the form `YYYYMMDDhhmmss`.</td>
        </tr>
        <tr>
            <td markdown="span">`evaluationOrder`</td>
            <td markdown="span">A value that determines the relative evaluation order of registered services (lower values come before higher values). This is especially important when more than one `serviceId` expression can match the same service; `evalutionOrder` deterines which expression is evaluated first.</td>
        </tr>
    </tbody>
</table>

## References

* [CAS 5: Service Management][casdoc-svc-mgmt]
* [CAS 5: JSON Service Registry][casdoc-svc-reg-json]

{% include reflinks.md %}
{% include links.html %}
