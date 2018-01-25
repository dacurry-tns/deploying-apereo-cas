---
title: Load the MongoDB service registry from the JSON service registry
last_updated: MoDecember 21, 2017
sidebar: main_sidebar
permalink: high-avail_service-registry_load_mongodb_from_json.html
summary:
---

If no other service registry is configured, CAS will use an in-memory service registry (not suitable for production deployments) and, to make it possible to experiment with the server, automatically initialize that registry from some default JSON service definitions included with the software (these are stored in the application classpath). But the auto-initialization functionality is actually more flexible than this: it can initialize any configured service registry, not just the in-memory service registry, and it can be told where to look for the set of JSON service definitions that it will populate into that registry. We will make use of these features to transfer the contents of the existing JSON service registry to the new MongoDB service registry.

## Configure service registry auto-initialization

Edit the file `etc/cas/config/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and locate the `cas.serviceRegistry.json.location` property (around line 7) that was added when we [set up the original service registry][building_server_service-registry_configure-the-service-registry]:

```properties
cas.serviceRegistry.json.location:     file:/etc/cas/services
```

(If you deleted this setting when adding the MongoDB settings in the previous section, add it back, because it's needed for this step.) Add the `cas.serviceRegistry.initFromJson` property to enable the automatice service registry initialization functionality:

```properties
cas.serviceRegistry.json.location:     file:/etc/cas/services
cas.serviceRegistry.initFromJson:      true
```

## Install and run on the master build server

Use the [updated build and installation scripts][building_svcmgmt_install-and-test-the-webapp] (or repeat the commands) to install the updated CAS server and services management webapp on the master build server (***casdev-master***) and restart Tomcat:

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# sh /opt/scripts/cassrv-install.sh
---Installing on casdev-master.newschool.edu
Installation complete.
casdev-master#  
```

Review the contents of the log files (`/var/log/tomcat/catalina.yyyy-mm-dd.out` and `/var/log/cas/cas.log`) for errors. This warning may appear in `cas.log`:

```
Runtime memory is used as the persistence storage for retrieving and persisting service definitions. Changes that
are made to service definitions during runtime WILL be LOST when the web server is restarted. Ideally for
production, you need to choose a storage option (JDBC, etc) to store and track service definitions.
```

and can be safely ignored (its appearance is a side effect caused by the fact that we haven't actually created anything in the MongoDB service registry yet).

## Verify that the MongoDB service registry was created and populated

Once the server has finished its startup, connect to MongoDB with the `mongo` shell and check to see that the service registry collection (`casServiceRegistry`) has been created, and that it has the appropriate contents. Use the MongoDB connection string to ensure that you get connected to the primary:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">casdev-master# </span><span class="nc">mongo</span><span class="kv"> 'mongodb://mongocas:changeit@casdev-srv01.newschool.edu,casdev-srv02.newschool.edu,casdev-srv03.newschool.edu/casdb?replicaSet=rs0&ssl=false'
</span>MongoDB shell version v3.6.0
connecting to: mongodb://mongocas:changeit@casdev-srv01.newschool.edu,casdev-srv02.newschool.edu,casdev-srv03.newschool.edu/casdb?replicaSet=rs0&ssl=false
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Starting new replica set monitor for rs0/casdev-srv01.newschool.edu:27017,casdev-srv02.newschool.edu:27017,casdev-srv03.newschool.edu:27017
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [ReplicaSetMonitor-TaskExecutor-0] Successfully connected to casdev-srv01.newschool.edu:27017 (1 connections now open to casdev-srv01.newschool.edu:27017 with a 5 second timeout)
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Successfully connected to casdev-srv02.newschool.edu:27017 (1 connections now open to casdev-srv02.newschool.edu:27017 with a 5 second timeout)
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [ReplicaSetMonitor-TaskExecutor-0] Successfully connected to casdev-srv03.newschool.edu:27017 (1 connections now open to casdev-srv03.newschool.edu:27017 with a 5 second timeout)
MongoDB server version: 3.6.0
<span class="ni">rs0:PRIMARY&gt; </span><span class="nc">show</span><span class="kv"> collections</span>
casServiceRegistry
proxyGrantingTicketsCollection
proxyTicketsCollection
samlArtifactsCache
samlAttributeQueryCache
serviceTicketsCollection
ticketGrantingTicketsCollection
<span class="ni">rs0:PRIMARY&gt; </span><span class="nc">db.casServiceRegistry.distinct(</span><span class="kv">"serviceId"</span><span class="nc">)</span>
[
        "https://casdev.newschool.edu/cas/idp/profile/SAML2/Callback.+",
        "^https://casdev-casapp.newschool.edu/secured-by-cas(\\z|/.*)",
        "https://casdev-samlsp.newschool.edu/shibboleth",
        "^https://casdev-casapp.newschool.edu/return-mapped(\\z|/.*)",
        "^https://casdev-casapp.newschool.edu/secured-by-cas-duo(\\z|/.*)",
        "^https://casdev.newschool.edu/cas/status/dashboard(\\z|/.*)",
        "^https://casdev.newschool.edu/cas-management(\\z|/.*)"
]
<span class="ni">rs0:PRIMARY&gt; </span>
</code></pre>
</div>

There should be one entry in the collection for each service defined in the JSON service registry.

## Shut down the application

Once the MongDB service registry has been initialized, shut down the CAS server by running the command

```console
casdev-master# systemctl stop tomcat
```

## Remove service registry auto-initialization settings

Edit the file `etc/cas/config/cas.properties` in the `cas-overlay-template` directory again and delete the `cas.serviceRegistry.json.location` and `cas.serviceRegistry.initFromJson` settings.

{% include reflinks.md %}
{% include links.html %}
