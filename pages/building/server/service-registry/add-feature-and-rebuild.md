---
title: Add the feature and rebuild the server
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_service-registry_add-feature-and-rebuild.html
summary:
---

Adding the JSON service registry feature requires adding a new dependency to the Maven project object model and rebuilding the server.

## Add the JSON service registry to the project object model

To add JSON service registry support to the CAS server, edit the file `pom.xml` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and locate the `dependencies` section (around line 61), which should look something like this:

```xml
<dependencies>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-webapp${app.server}</artifactId>
        <version>${cas.version}</version>
        <type>war</type>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

Insert a new dependency for the JSON service registry:

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

## Rebuild the server

Run Maven again to rebuild the server according to the new model:

```console
casdev-master# ./mvnw clean package
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building cas-overlay 1.0
[INFO] ------------------------------------------------------------------------
(lots of diagnostic output...check for errors)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:07 min
[INFO] Finished at: YYYY-MM-DDTHH:MM:SS-00:00
[INFO] Final Memory: 25M/70M
[INFO] ------------------------------------------------------------------------
casdev-master#  
```

## References

* [CAS 5: JSON Service Registry][casdoc-svc-reg-json]

{% include reflinks.md %}
