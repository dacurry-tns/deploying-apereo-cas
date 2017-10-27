---
title: Configure the SP
last_updated: October 27, 2017
sidebar: main_sidebar
permalink: building_samlclient_configure-the-sp.html
summary:
---

Now that the Shibboleth SP has been installed, the `shibd` daemon can be configured to communicate with the CAS SAML IdP.

{% include note.html content="The steps in this section should be performed on the client server (***casdev-samlsp***), not the master build server (***casdev-master***)." %}

## Configure the SAML entity and IdP settings

Edit the file `/etc/shibboleth/shibboleth2.xml` and make the changes described below to set the SP's *entityID* (the string the SP uses to identify itself to the IdP) and tell it which IdP to use.

### Set the entityID

Locate the `<ApplicationDefaults>` XML tag (around line 23) and change the value of the `entityID` attribute to reflect the URL of the SAML client host (***casdev-samlsp.newschool.edu***).

```xml
<ApplicationDefaults entityID="https://casdev-samlsp.newschool.edu/shibboleth"
                     REMOTE_USER="eppn persistent-id targeted-id">
```

### Set the `REMOTE_USER` attribute and attribute prefix

On the next line, change the value of the `REMOTE_USER` attribute to `uid`, and add a new attribute, `attributePrefix`, as shown:

```xml
<ApplicationDefaults entityID="https://casdev-samlsp.newschool.edu/shibboleth"
                     REMOTE_USER="uid" attributePrefix="SAML_">
```

The `REMOTE_USER` attribute specifies which user attribute, returned by the IdP, should be used to populate the `REMOTE_USER` environment variable for the web application to access. The `attributePrefix` attribute specifies a prefix string to be applied to all the environment variables set by the `mod_shib` plugin, including the environment variables containing user attribute values.

### Configure session security

Locate the `<Sessions>` XML tag (around line 35) and make sure the value of the `handlerSSL` attribute is set to `true`, and the value of the `cookieProps` attribute is set to `https`:

```xml
<Sessions lifetime="28800" timeout="3600" relayState="ss:mem"
          checkAddress="false" handlerSSL="true" cookieProps="https">
```

These settings will ensure that all sessions between the SP and the IdP are encrypted with TLS/SSL, and that cookies cannot be exchanged over insecure channels.

### Point the SP to the IdP

Locate the `<SSO>` XML tage (around line 44) and change the value of the `entityID` attribute to the URL of the CAS SAML IdP. Delete the `discoveryProtocol` and `discoveryURL` attributes; they are not needed for this configuration.

```xml
<SSO entityID="https://casdev.newschool.edu/cas/idp">
  SAML2 SAML1
</SSO>
```

### Tell the SP where to get the IdP's metadata

Locate the (commented out) examples of `MetadataProvider` definitions (around lines 73-90), and insert the following below them:

```xml
<MetadataProvider type="XML" validate="true"
      uri="https://casdev.newschool.edu/cas/idp/metadata"
      backingFilePath="casdev-metadata.xml" reloadInterval="7200">
</MetadataProvider>
```

This tells the SP what URL to use to obtain the IdP's metadata, gives it a file name in which to store it, and a time limit after which it should be reloaded from the server. (The metadata backing file will be stored in `/var/cache/shibboleth`.)

## Configure attribute processing

A SAML IdP sends user attributes to a SAML SP in the form of SAML assertions. To avoid misinterpretation, every attribute has a unique identifier, agreed upon by standards-setting bodies. This identifier (name) is different from the attribute names used by back-end data stores and consuming applications. For example, a telephone number might be returned from the IdP as follows:

```xml
<saml:Attribute FriendlyName="telephoneNumber" Name="urn:oid:2.5.4.20"
    NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
    <saml:AttributeValue xmlns:xs="http://www.w3.org/2001/XMLSchema"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:type="xs:string">555-5555</saml:AttributeValue>
</saml:Attribute>
```

Although the back-end data store from which the IdP obtained the attribute (e.g., an LDAP directory) might refer to this attribute as a `telephoneNumber`, a consuming application might call it `telephoneNumber` or `telephone` or `phone` or something else. Therefore, to make sure that IdPs and SPs know that they're talking about the same thing, the [SAML V2.0 LDAP/X.500 Attribute Profile][https://wiki.oasis-open.org/security/SstcSaml2AttributeX500Profile] specifies that this attribute should be identified as `urn:oid:2.5.4.20` (similar values are defined for other common LDAP attributes).

To map between the standard attribute names used by the IdP and SP and the "friendly" attribute names used by applications, the Shibboleth SP uses a file called `/etc/shibboleth/attribute-map.xml`, which contains definitions like this:

```xml
<Attribute name="urn:oid:2.5.4.20" id="telephoneNumber"/>
<Attribute name="urn:mace:dir:attribute-def:telephoneNumber" id="telephoneNumber"/>
```

The `urn:mace` attribute namespace is another namespace, registered with the IETF and IANA, for Internet2's Middleware Architecture Committee for Education. It is heavily used by Internet2 and InCommon member organizations.

### Enable LDAP attribute mappings

To enable the pre-defined LDAP attribute mappings in the SP, edit the file `/etc/shibboleth/attribute-map.xml` and remove the comment lines (`<--` and `-->`) around the section labeled "Examples of LDAP-based attributes" (around lines 92-149).

### Add custom attribute mappings

When we [configured attribute resolution][building_server_ldap_resolution-release_configure-attribute-resolution] in the CAS server, we configured a number of standard LDAP attributes, but also a couple of non-standard ones, `role` and `UDC_IDENTIFIER`. To tell the Shibboleth SP how to process these attributes, edit the file `/etc/shibboleth/attrbute-map.xml` and add the following lines to the end (before the `</Attributes>` XML close tag):

```xml
<Attribute name="urn:newschool:attribute-def:role" id="role"/>
<Attribute name="urn:newschool:attribute-def:UDC_IDENTIFIER" id="UDC_IDENTIFIER"/>
```

We cannot use the `urn:oid` or `urn:mace` namespaces, since those are controlled by standards bodies. So instead, we define our own namespace, `urn:newschool`, modeled after `urn:mace`.

## Restart `shibd`

Run the command

```console
casdev-samlsp# systemctl restart shibd
```

to restart the Shibboleth daemon with the new configuration.

## References

* [Shibboleth SP: NativeSPConfiguration][shibboleth-spconfig]
* [ShibInstallFest: Linux Service Provider (RHEL 7.0)][shibinstallfest]
* [Shibboleth: AttributeNaming][shibboleth-attr-naming]

{% include reflinks.md %}
{% include links.html %}
