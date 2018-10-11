---
title: Update the server configuration
last_updated: October 11, 2018
sidebar: main_sidebar
permalink: high-avail_service-registry_update-the-server-configuration.html
summary:
---

Enabling the MongoDB service registry requires updating a dependency in the project object model, rebuilding the server, and updating configuration properties. The same steps must be performed on the management webapp, which also works with the service registry.

## Update the dependency in the project model

As with all other features of the CAS server (and the management webapp), the MongoDB service registry is enabled by adding a new dependency to the project object model (`pom.xml`). But this time we are actually replacing one feature (the JSON service registry) with another, so rather than adding a new dependency to the list, we will update one in place.

### CAS server

Edit the file `pom.xml` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and locate the `cas-server-support-json-service-registry` dependency (around line 79) that was added when the service registry was [first enabled][building_server_service-registry_add-feature-and-rebuild]], which should look like this:

```xml
<dependencies>
    ....
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-json-service-registry</artifactId>
        <version>${cas.version}</version>
    </dependency>
    ....
</dependencies>
```

Replace the `json` in the dependency name with `mongo`, so that it now looks like this:

```xml
<dependencies>
    ....
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-mongo-service-registry</artifactId>
        <version>${cas.version}</version>
    </dependency>
    ....
</dependencies>
```

This will instruct Maven to download the appropriate code modules and build them into the server.

### Service management webapp

Edit the file `pom.xml` in the `cas-management-overlay` directory on the master build server (***casdev-master***) and locate the `cas-server-support-json-service-registry` dependency (around line 78) that was added when the management webapp was [first built][building_svcmgmt_build-the-webapp], and repeat the change described above (replace `json` with `mongo` in the name of the dependency).

## Rebuild the application

Run Maven to rebuild the applications according to the new models.

### CAS server

From the `cas-overlay-template` directory:

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
[INFO] Total time: 25.411 s
[INFO] Finished at: YYYY-MM-DDTHH:MM:SS-00:00
[INFO] Final Memory: 37M/96M
[INFO] ------------------------------------------------------------------------
casdev-master#  
```

### Manaagement webapp

From the `cas-management-overlay` directory:

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
[INFO] Total time: 8.229 s
[INFO] Finished at: YYYY-MM-DDTHH:MM:SS-00:00
[INFO] Final Memory: 30M/72M
[INFO] ------------------------------------------------------------------------
casdev-master#  
```

## Configure MongoDB service registry properties

Both the CAS server and the management webapp need to be configured to recognize and use the MongoDB service registry.

### CAS server

Add the following settings to `etc/cas/config/cas.properties` in the `cas-overlay-template` directory:

```properties
#
# Components of the MongoDB connection string broken out for ease of editing.
# See https://docs.mongodb.com/manual/reference/connection-string/
#
mongo.db:                               casdb
mongo.rs:                               rs0
mongo.opts:                             &ssl=false
mongo.creds:                            mongocas:changeit
mongo.hosts:                            casdev-srv01-lid.newschool.edu,casdev-srv02-lid.newschool.edu,casdev-srv03-lid.newschool.edu

#
# The connection string, assembled
#
mongo.uri:                              mongodb://${mongo.creds}@${mongo.hosts}/${mongo.db}?replicaSet=${mongo.rs}${mongo.opts}

#
# Service registry
#
cas.serviceRegistry.mongo.clientUri:    ${mongo.uri}
cas.serviceRegistry.mongo.collection:   casServiceRegistry
```

The `cas.serviceRegistry.mongo.clientUri` property is the only setting specific to the service registry that needs to be set; it takes a MongoDB connection string as a value. The format of a MongoDB connection string is:

```
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```

The `mongo.*` properties are optional; they make it easier to edit the components of the connection string, and  also make it easier to repeat the connection string in other settings (such as for the [MongoDB ticket registry][high-avail_ticket-registry_overview]) without having to duplicate values that may change over time.

The `cas.serviceRegistry.mongo.collection` property setting is not required, but is recommended. By default, CAS calls the collection `cas-service-registry`. This is a valid MongoDB collection name, but the `mongo` shell does not accept collection names that contain hyphens in some commands. Although it's possible to work around that using alternative command syntax, it's easier to just avoid the problem altogether by using a collection name that doesn't contain hyphens.

Do not delete the `cas.serviceRegistry.json.location` property from `cas.properties` just yet; it's still needed to transfer the contents of the JSON service registry to the MongoDB service registry.

### Management webapp

Copy the settings that were added to the CAS server above to `etc/cas/config/management.properties` in the `cas-management-overlay` directory.

Delete the `cas.serviceRegistry.json.location` property from `management.properties`; it is not needed now that we are using the MongoDB service registry.

## References

* [CAS 5: Mongo Service Registry][casdoc-svc-reg-mongo]
* [CAS 5: Configuration Properties: MongoDb Service Registry][casdoc-config-mongo-sr]

{% include reflinks.md %}
{% include links.html %}
