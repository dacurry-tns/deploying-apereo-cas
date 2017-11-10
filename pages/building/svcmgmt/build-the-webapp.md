---
title: Build the webapp
last_updated: November 8, 2017
sidebar: main_sidebar
permalink: building_svcmgmt_build-the-webapp.html
summary:
---

Before building the services management webapp, the build must be configured to include the same service registry persistence method as the CAS server. This is accomplished by adding a dependency to the Maven project object model. For this project, that means adding the JSON service registry.

## Add the JSON service registry to the project object model

Just as we [did for the CAS server][building_server_service-registry_add-feature-and-rebuild], edit the file `pom.xml` in the `cas-services-management-overlay` directory on the master build server (***casdev-master***) and locate the `dependencies` section (around line 69), which should look something like this:

```xml
<dependencies>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-management-webapp</artifactId>
        <version>${cas.version}</version>
        <type>war</type>
    </dependency>
</dependencies>
```

Insert a new dependency for the JSON service registry:

```xml
<dependencies>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-management-webapp</artifactId>
        <version>${cas.version}</version>
        <type>war</type>
    </dependency>
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-json-service-registry</artifactId>
        <version>${cas.version}</version>
    </dependency>
</dependencies>
```

## Build the webapp

Run Maven to build the webapp according to the project object model:

```console
casdev-master# ./mvnw clean package
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building cas-services-management-overlay 1.0
[INFO] ------------------------------------------------------------------------
(lots of diagnostic output...check for errors)
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 25.841 s
[INFO] Finished at: YYYY-MM-DDTHH:MM:SS-00:00
[INFO] Final Memory: 29M/72M
[INFO] ------------------------------------------------------------------------
casdev-master#  
```

## References

* [CAS 5: Services Management Webapp][casdoc-svc-mgmt-webapp]
* [CAS 5: JSON Service Registry][casdoc-svc-reg-json]

{% include reflinks.md %}
{% include links.html %}
