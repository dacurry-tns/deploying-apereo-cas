---
title: Update the service registry
last_updated: November 2, 2017
sidebar: main_sidebar
permalink: building_samlclient_update-the-service-registry.html
summary:
---

Just like CAS-enabled services, SAML-enabled services must be defined in the service registry.

## Create a service definition for the SAML client

Create the file `etc/cas/services/ApacheSecuredBySAML-1509030300.json` (replace `1509030300` with the current `date +%s` or `YYYYMMDDhhmmss` value) in the `cas-overlay-template` directory on the master build server (***casdev-master***) with the following contents:

```json
{
  "@class" : "org.apereo.cas.support.saml.services.SamlRegisteredService",
  "serviceId" : "https://casdev-samlsp.newschool.edu/shibboleth",
  "name" : "Apache Secured By SAML",
  "id" : 1509030300,
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
  "evaluationOrder" : 1125
}
```

This is similar to the service definitions created previously, but there are some differences:

1. The `@class` of the service is `org.apereo.cas.support.saml.services.SamlRegisteredService` rather than `org.apereo.cas.services.RegexRegisteredService`.
2. The `serviceId` is specified as an exact-match string, not a regular expression. Specifically, this attribute must be equal to the entityID of the service.
3. The new attribute, `metadataLocation`, is used to tell the IdP where it can obtain the SP's metadata. This will be automatically retrieved and stored in `/etc/cas/saml/metadata-backups/` when the SP first connects to the IdP. For SPs that do not provide a URL from which to obtain metadata, the metadata can be obtained by other means (e.g., email from the service provider) and saved to a file which can then be identified in this attribute with a `file:` URI.
4. The `ReturnMappedAttributeReleasePolicy` is used to assign the SAML-specific attribute names expected by the SP to the attributes. The values to be used here can be obtained from the SP's `/etc/shibboleth/attribute-map.xml` file.

## References

* [CAS 5: JSON Service Registry][casdoc-svc-reg-json]
* [CAS 5: SAML2 Authentication][casdoc-config-saml2-auth]

{% include reflinks.md %}
