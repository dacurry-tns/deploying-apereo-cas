---
title: Update the service registry
last_updated: September 29, 2017
sidebar: main_sidebar
permalink: building_server_ldap_resolution-release_update-the-service-registry.html
summary:
---

Attribute release policies are defined on a per-service basis in the service registry. There are four basic attribute release policies:

<table>
    <colgroup>
        <col width="25%" />
        <col width="75%" />
    </colgroup>
    <tbody>
        <tr>
            <td markdown="span">**Return All**</td>
            <td markdown="span">Return all resolved attributes to the service.</td>
        </tr>
        <tr>
           <td markdown="span">**Deny All**</td>
           <td markdown="span">Do not return any attributes to the service. This will also prevent the release of the default attribute pool (see the note below).</td>
        </tr>
        <tr>
            <td markdown="span">**Return Allowed**</td>
            <td markdown="span">Only return the attributes specifically allowed by the policy. This policy includes a list of the attributes to release.</td>
        </tr>
        <tr>
            <td markdown="span">**Return Mapped**</td>
            <td markdown="span">Only return the attributes specifically allowed by the policy, but also allow them to be renamed at the individual service level. Useful when a particular service insists on having specific attribute names not used by other services.</td>
        </tr>
    </tbody>
</table>

The syntax for defining the above policies is defined in the CAS 5 *Attribute Release Policies* documentation. That document also describes a number of script-based policies that will call a Groovy, JavaScript, or Python script to decide how to release attributes (these policies are beyond the scope of this document).

{% include note.html content="The `cas.authn.attributeRepository.defaultAttributesToRelease` property can be set in `cas.properties` to a comma-separated list of attributes that should be released to all services, without having to list them in every service definition. We are not using this feature in our installation, because it makes it harder to determine which attributes are released to a particular service (by requiring the administrator to look in more than one location)." %}

## Create a "return all attributes" service definition for the CAS client

When we initially [created the service registry][building_server_service-registry_configure-the-service-registry], we created a wildcard service definition that would match any service. Now however, it makes sense to create a specific definition for our CAS client, and use that definition to release attributes to the client. Create a file in the `etc/cas/services` directory on the master build server (***casdev-master***) with the following contents:

```json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^https://casdev-casapp.newschool.edu/secured-by-cas(\\z|/.*)",
  "name" : "Apache Secured By CAS",
  "id" : 201700830155400,
  "description" : "CAS development Apache mod_auth_cas server with username/password protection",
  "attributeReleasePolicy" : {
    "@class" : "org.apereo.cas.services.ReturnAllAttributeReleasePolicy"
  },
  "evaluationOrder" : 1100
}
```

Following the naming convention introduced [earlier][building_server_service-registry_configure-the-service-registry], the name of this file should be `ApacheSecuredByCAS-201700830155400.json`. (Your `id`, and therefore that part of the filename, should use the current value for `YYYYMMDDhhmmss`.)

This service definition uses a `serviceId` regular expression that matches only the URL for the `secured-by-cas` directory on the ***casdev-casapp*** server. The `(\\z|/.*)` syntax at the end matches either the empty string (`\\z`) or a slash ('/') followed by anything (`/.*`), meaning that the following will match:

```
https://casdev-casapp.newschool.edu/secured-by-cas
https://casdev-casapp.newschool.edu/secured-by-cas/
https://casdev-casapp.newschool.edu/secured-by-cas/index.php
https://casdev-casapp.newschool.edu/secured-by-cas/subdir/file.html
```

but the following will not:

```
https://casdev-casapp.newschool.edu/secured-by-cas-and-something-else
https://casdev-casapp.newschool.edu/some/other/path
https://casdev-master.newschool.edu/secured-by-cas
```

This service definition uses a `description` property instead of a comment to describe the service; this way the definition will appear in the [service management webapp][building_svcmgmt_overview].

The `evaluationOrder` has been given a value lower than that of the wildcard definition, so this definition will be matched first.

And finally, this definition includes the "Release All" `attributeReleasePolicy` property, which means that the CAS client will receive all attributes that could be resolved for the authenticating user.

## Create a "return mapped attributes" service definition for the CAS client

One of the applications that we use, Ellucian's Luminis portal, expects to receive a couple of attributes with names other than the ones commonly used: instead of a `mail` attribute, it expects to receive an `EmailAddress` attribute, and instead of a `givenName` attribute, it expects to receive a `Formatted Name` attribute (despite the fact that attribute names are not supposed to contain spaces). We could have made these mappings in the `cas.properties` file, but that would then require all applications to support these unusual attribute names. So instead, we will use the "Return Mapped" attribute release policy to perform the mapping only for this application.

To test this idea with our CAS client, create a file in the `etc/cas/services` directory on the master build server (***casdev-master***) with the following contents:

```json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^https://casdev-casapp.newschool.edu/return-mapped(\\z|/.*)",
  "name" : "Return Mapped Test",
  "id" : 20170927092000,
  "description" : "Display results of a Return Mapped attribute release policy",
  "attributeReleasePolicy" : {
    "@class" : "org.apereo.cas.services.ReturnMappedAttributeReleasePolicy",
    "allowedAttributes" : {
      "@class" : "java.util.TreeMap",
      "cn" : "cn",
      "displayName" : "displayName",
      "givenName" : "Formatted Name",
      "mail" : "EmailAddress",
      "memberOf" : "memberOf",
      "role" : "role",
      "sn" : "sn",
      "uid" : "uid",
      "UDC_IDENTIFIER": "UDC_IDENTIFIER"
    }
  },
  "evaluationOrder" : 1150
}
```

The name of this file should be `ReturnMappedTest-20170927092000.json`. (Your `id`, and therefore that part of the filename, should use the current value for `YYYYMMDDhhmmss`.)

In this definition, the `attributeReleasePolicy` property uses the `ReturnMappedAttributeReleasePolicy` instead of the `ReturnAllAttributeReleasePolicy`; this requires us to provide a new sub-property called `allowedAttributes` that contains the list of attributes to be released. For each attribute, the attribute's name (as set in `cas.properties`) appears on the left, and the name it should be released with (the *mapped* name) *for this application only* appears on the right.

## References

* [CAS 5: Attribute Release Policies][casdoc-attrib-rel-pol]
* [CAS 5: JSON Service Registry][casdoc-svc-reg-json]

{% include reflinks.md %}
{% include links.html %}
