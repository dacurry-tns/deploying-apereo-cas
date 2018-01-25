---
title: Update the server configuration
last_updated: December 21, 2017
sidebar: main_sidebar
permalink: high-avail_ticket-registry_update-the-server-configuration.html
summary:
---

Enabling the MongoDB ticket registry requires adding a new dependency to the Maven project object model, rebuilding the server, and updating `cas.properties`.

## Add the dependency to the project model

To add the MongoDB ticket registry to the CAS server, edit the file `pom.xml` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and locate the dependencies section (around line 69), which should look something like this:

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

Insert the new dependency at the bottom of the section:

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
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-mongo-ticket-registry</artifactId>
        <version>${cas.version}</version>
    </dependency>
</dependencies>
```

This will instruct Maven to download the appropriate code modules and build them into the server.

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
[INFO] Total time: 17.257 s
[INFO] Finished at: YYYY-MM-DDTHH:MM:SS-00:00
[INFO] Final Memory: 39M/97M
[INFO] ------------------------------------------------------------------------
casdev-master#  
```

## Configure MongoDB ticket registry properties

Add the following settings to `etc/cas/config/cas.properties` in the `cas-overlay-template` directory:

```properties
#
# Components of the MongoDB connection string broken out for ease of editing.
# See https://docs.mongodb.com/reference/connection-string/
#
mongo.db:                              casdb
mongo.rs:                              rs0
mongo.opts:                            &ssl=false
mongo.creds:                           mongocas:changeit
mongo.hosts:                           casdev-srv01.newschool.edu,casdev-srv02.newschool.edu,casdev-srv03.newschool.edu

#
# The connection string, assembled
#
mongo.uri:                             mongodb://${mongo.creds}@${mongo.hosts}/${mongo.db}?replicaSet=${mongo.rs}${mongo.opts}

#
# Ticket registry
#
cas.ticket.registry.mongo.clientUri:   ${mongo.uri}
```

The `cas.ticket.registry.mongo.clientUri` property is the only setting specific to the ticket registry that needs to be set; it takes a MongoDB connection string as a value. The format of a MongoDB connection string is:

```
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```

The `mongo.*` properties are optional; they make it easier to edit the components of the connection string, and will also make it easier to repeat the connection string in other settings (such as for the [MongoDB service registry][high-avail_service-registry_overview]) without having to duplicate values that may change over time.

## Enable MongoDB Java driver logging

The Log4J configuration file included with the Maven WAR overlay template does not include a logger for the MongoDB Java driver, which is what the CAS server uses to communicate with MongoDB. To enable MongoDB Java driver logging, edit the file `etc/cas/config/log4j2.xml` on the master build server (***casdev-master***) and locate the bottom of the list of `AsyncLogger` definitions (around line 97), which should look something like this:

```xml
<AsyncLogger name="org.ldaptive" level="warn" />
<AsyncLogger name="com.hazelcast" level="warn" />
<AsyncLogger name="org.jasig.spring" level="warn" />

<!-- Log perf stats only to perfStats.log -->
```

Add a new definition for `org.mongodb.driver` to the list:

```xml
<AsyncLogger name="org.ldaptive" level="warn" />
<AsyncLogger name="com.hazelcast" level="warn" />
<AsyncLogger name="org.jasig.spring" level="warn" />
<AsyncLogger name="org.mongodb.driver" level="warn" />

<!-- Log perf stats only to perfStats.log -->
```

Although this line is not critical to the regular operation of the CAS server (warnings and errors will just propagate upward), it is immensely helpful when trying to debug MongoDB connection problems (set `level` to `debug`).

## References

* [CAS 5: MongoDB Ticket Registry][casdoc-mongo-ticket-reg]
* [CAS 5: Configuration Properties: MongoDB Ticket Registry][casdoc-config-mongo-tr]
* [MongoDB: Connection String URI Format][mongodb-conn-string]

{% include reflinks.md %}
