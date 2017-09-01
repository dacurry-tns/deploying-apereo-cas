---
title: Configuring LDAP authentication
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_ldap_authentication_overview.html
summary: The LDAP module will be added to the CAS server to enable it to authenticate users against LDAP directories
---

The CAS server's LDAP integration enables the server to authenticate users against LDAP directories such as Active Directory and the LDAP directory included with Ellucian's Luminis user portal.

## Add the LDAP dependency to the project object model

To add LDAP support to the CAS server, edit the file `pom.xml` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and locate the dependencies section (around line 61), which should look something like this:

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
</dependencies>
```

Insert a new dependency for the LDAP module:

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
[INFO] Total time: 9.368 s
[INFO] Finished at: YYYY-MM-DDTHH:MM:SS-00:00
[INFO] Final Memory: 27M/78M
[INFO] ------------------------------------------------------------------------
casdev-master#  
```

## Disable use of built-in credentials

Until they're disabled, CAS will use the built-in username and password (`casuser` / `Mellon`) regardless of what other authentication methods have been configured. To disable the built-in credentials, add the following line to `etc/cas/config/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***):

```properties
cas.authn.accept.users:
```

## Commit changes to Git

Commit the changes made to `pom.xml` and `cas.properties` to Git to make them easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add etc/cas/config/cas.properties
casdev-master# git add pom.xml
casdev-master# git commit -m "Added LDAP support"
[newschool-casdev 2dd8813] Added LDAP support
 2 files changed, 10 insertions(+)
casdev-master#  
```

on the master build server (***casdev-master***).

## References

* [CAS 5: LDAP Authentication][casdoc-ldap-auth]
* [CAS 5: Configuration Properties: LDAP Authentication][casdoc-ldap-auth-props]

{% include reflinks.md %}
