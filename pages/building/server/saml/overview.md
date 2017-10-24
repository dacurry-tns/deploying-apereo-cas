---
title: Adding SAML support
last_updated: October 23, 2017
sidebar: main_sidebar
permalink: building_server_saml_overview.html
summary: CAS 5's native capability to operate as a SAML2 Identity Provider will be used to provide authentication and single sign-on support to services that do not support the CAS protocol.
---

The CAS protocol is our preferred protocol for authentication and single sign-on: it's easy to understand, easy to configure, and "just works." Unfortunately, while CAS is pretty well supported by applications and services designed for the higher education market, is is much less widely supported by applications and services targeted mainly at the corporate world. These applications and services typically use the SAML2 protocol instead. Prior to CAS 5, supporting both protocols together required setting up a [Shibboleth][shibboleth-idp] server alongside the CAS server, and configuring one server to use the other as its authentication source. Although this more or less worked, it was difficult to manage, and didn't handle all aspects of single sign-on (especially single log-out) very cleanly.

Fortunately, CAS 5 has added full native SAML2 support, allowing it to play the role of a SAML2 Identity Provider (IdP). The IdP is the back-end server that SAML2 clients (Service Providers, or SPs) communicate with to authenticate users, retrieve attributes, etc. We will use this new functionality to add SAML2 support to our single sign-on environment while at the same time eliminating our dependency on Shibboleth.

## Add the SAML2 IdP dependency to the project object model

To add SAML2 IdP support to the CAS server, edit the file `pom.xml` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and locate the dependencies section (around line 69), which should look something like this:

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
    <dependency>
         <groupId>org.apereo.cas</groupId>
         <artifactId>cas-server-support-duo</artifactId>
         <version>${cas.version}</version>
    </dependency>
</dependencies>
```

Insert the new dependency for the SAML2 IdP support just after the `cas-server-support-saml` dependency added previously:

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
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-saml-idp</artifactId>
        <version>${cas.version}</version>
    </dependency>
    <dependency>
         <groupId>org.apereo.cas</groupId>
         <artifactId>cas-server-support-duo</artifactId>
         <version>${cas.version}</version>
    </dependency>
</dependencies>
```

This will instruct Maven to download the appropriate code modules and build them into the server.

{% include note.html content="There are two different SAML dependencies: `cas-server-support-saml` enables support for returning user attributes to client applications; `cas-server-support-saml-idp` enables support for using the CAS server as a SAML2 Identity Provider. Both dependencies should be included in `pom.xml`." %}

## Rebuild the server

Run Maven to rebuild the server according to the new model:

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
[INFO] Total time: 01:00 min
[INFO] Finished at: YYYY-MM-DDTHH:MM:SS-00:00
[INFO] Final Memory: 35M/84M
[INFO] ------------------------------------------------------------------------
casdev-master#  
```

## References

* [CAS 5: Configuring SAML2 Authentication][casdoc-config-saml2-auth]

{% include reflinks.md %}
{% include links.html %}
