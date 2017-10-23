---
title: Configuring LDAP attribute resolution and release
last_updated: October 23, 2017
sidebar: main_sidebar
permalink: building_server_ldap_resolution-release_overview.html
summary: To enable client applications to obtain information about authenticated users, the CAS server must be configured to resolve attributes and release them to the clients.
---

Version 3 of the CAS protocol, which was first supported by CAS 4.0, contains native support for returning authentication/user attributes to clients. Version 2 of the CAS protocol, the version implemented by CAS 3.*x*, did not support attribute release; the SAML 1.1 protocol was used for that purpose. Most CAS clients have not yet been updated to support Version 3 of the protocol, so it's still necessary to configure SAML 1.1-based attribute release.

## Add the SAML 1.1 dependency to the project object model

To add SAML 1.1 support to the CAS server, edit the file `pom.xml` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and locate the dependencies section (around line 69), which should look something like this:

```xml
<dependencies>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-webapp${app.server}</artifactId>
        <version>${cas.version}</version>
        <type>war</type>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-json-service-registry</artifactId>
        <version>${cas.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-ldap</artifactId>
        <version>${cas.version}</version>
    </dependency>
</dependencies>
```

Insert a new dependency for the SAML 1.1 module:

```xml
<dependencies>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-webapp${app.server}</artifactId>
        <version>${cas.version}</version>
        <type>war</type>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-json-service-registry</artifactId>
        <version>${cas.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-ldap</artifactId>
        <version>${cas.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-saml</artifactId>
        <version>${cas.version}</version>
    </dependency>
</dependencies>
```

This will instruct Maven to download the appropriate code modules and build them into the server.

## Rebuild the server

Run Maven again to rebuild the server according to the new model:

```console
casdev-master# ./mvnw clean package
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building cas-overlay 1.0
[INFO] ------------------------------------------------------------------------
(lots of diagnostic output... check for errors)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 43.966 s
[INFO] Finished at: YYYY-MM-DDTHH:MM:SS-00:00
[INFO] Final Memory: 30M/76M
[INFO] ------------------------------------------------------------------------
casdev-master#  
```

## References

* [CAS 5: SAML Protocol][casdoc-saml-protocol]

{% include reflinks.md %}
