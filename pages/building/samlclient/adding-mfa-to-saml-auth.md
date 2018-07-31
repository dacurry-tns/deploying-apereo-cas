---
title: Adding MFA to SAML authentication
last_updated: July 31, 2018
sidebar: main_sidebar
permalink: building_samlclient_adding-mfa-to-saml-auth.html
summary:
---

Adding a multi-factor authentication flow to a SAML-authenticated service is as easy as editing the service registry definition to add the `multifactorPolicy` directive. For example, the service registry definition file created in this chapter would look like this with Duo multi-factor authentication added:

```json
{
  "@class" : "org.apereo.cas.support.saml.services.SamlRegisteredService",
  "serviceId" : "https://casdev-samlsp.newschool.edu/shibboleth",
  "name" : "Apache Secured By SAML",
  "id" : 20171026110500,
  "description" : "CAS development Apache mod_shib/shibd server with username/password protection",
  "metadataLocation" : "https://casdev-samlsp.newschool.edu/Shibboleth.sso/Metadata",
  "attributeReleasePolicy" : {
    "@class" : "org.apereo.cas.services.ReturnMappedAttributeReleasePolicy",
    "allowedAttributes" : {
      "@class" : "java.util.TreeMap",
      "cn" : "urn:oid:2.5.4.3",
      "displayName" : "urn:oid:2.16.840.1.113730.3.1.241",
      "givenName" : "urn:oid:2.5.4.42",
      "mail" : "urn:oid:0.9.2342.19200300.100.1.3",
      "role" : "urn:newschool:attribute-def:role",
      "sn" : "urn:oid:2.5.4.4",
      "uid" : "urn:oid:0.9.2342.19200300.100.1.1",
      "UDC_IDENTIFIER": "urn:newschool:attribute-def:UDC_IDENTIFIER"
    }
  },
  "multifactorPolicy" : {
    "@class" : "org.apereo.cas.services.DefaultRegisteredServiceMultifactorPolicy",
    "multifactorAuthenticationProviders" : [ "java.util.LinkedHashSet", [ "mfa-duo" ] ]
  },
  "evaluationOrder" : 1125
}
```

## Testing SAML and MFA together

In the service registry, CAS-enabled services are identified by URL. Thus, we were able to create CAS-only and CAS-plus-MFA services on the CAS client server simply by creating different directories in `/var/www/html`, resulting in two different URLs and two different service registry definitions, one with MFA enabled and one without.

SAML services are a little different, though. They're identified by an entityID, and generally there's only one entityID per application (recall that this value was set in the SP's configuration file, `/etc/shibboleth/shibboleth2.xml`). Thus, simply creating a second directory in `/var/www/html` won't work in and of itself, because the SP software will present the same entityID to the CAS SAML IdP for both directories. It *is* possible to configure the Shibboleth SP to present different entityID values under different conditions (such as different directories), but doing so is a complicated, multi-step undertaking that frankly isn't worth the effort.

Instead, to test SAML and MFA together, just add the `multifactorPolicy` attribute to the existing service definition and test. To disable MFA, just comment that part of the definition out:

```json
/*
"multifactorPolicy" : {
  "@class" : "org.apereo.cas.services.DefaultRegisteredServiceMultifactorPolicy",
  "multifactorAuthenticationProviders" : [ "java.util.LinkedHashSet", [ "mfa-duo" ] ]
},
*/
```

## References

* [CAS 5: Duo Security Authentication][casdoc-mfa-duo]
* [CAS 5: Configuration Properties: Multi-factor Authentication][casdoc-mfa-props]

{% include reflinks.md %}
{% include links.html %}
