---
title: Install and test the application
last_updated: December 21, 2017
sidebar: main_sidebar
permalink: high-avail_ticket-registry_install-and-test-the-application.html
summary:
---

The MongoDB ticket registry can be tested by installing the updated CAS software on the CAS servers and restarting it, and then accessing the test clients from multiple browsers.

## Install and test on the master build server

Use the [updated build and installation scripts][building_svcmgmt_install-and-test-the-webapp] (or repeat the commands) to install the updated CAS server on the master build server (***casdev-master***) and restart Tomcat:

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# sh /opt/scripts/cassrv-install.sh
---Installing on casdev-master.newschool.edu
Installation complete.
casdev-master#  
```

Review the contents of the log files (`/var/log/tomcat/catalina.yyyy-mm-dd.out` and `/var/log/cas/cas.log`) for errors. Then connect to MongoDB with the `mongo` shell and check to see that the ticket registry collections have been created. Use the MongoDB connection string to ensure that you get connected to the primary:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">casdev-master# </span><span class="nc">mongo</span><span class="kv"> 'mongodb://mongocas:changeit@casdev-srv01.newschool.edu,casdev-srv02.newschool.edu,casdev-srv03.newschool.edu/casdb?replicaSet=rs0&ssl=false'
</span>MongoDB shell version v3.6.0
connecting to: mongodb://mongocas:changeit@casdev-srv01.newschool.edu,casdev-srv02.newschool.edu,casdev-srv03.newschool.edu/casdb?replicaSet=rs0&ssl=false
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Starting new replica set monitor for rs0/casdev-srv01.newschool.edu:27017,casdev-srv02.newschool.edu:27017,casdev-srv03.newschool.edu:27017
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [ReplicaSetMonitor-TaskExecutor-0] Successfully connected to casdev-srv01.newschool.edu:27017 (1 connections now open to casdev-srv01.newschool.edu:27017 with a 5 second timeout)
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Successfully connected to casdev-srv02.newschool.edu:27017 (1 connections now open to casdev-srv02.newschool.edu:27017 with a 5 second timeout)
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [ReplicaSetMonitor-TaskExecutor-0] Successfully connected to casdev-srv03.newschool.edu:27017 (1 connections now open to casdev-srv03.newschool.edu:27017 with a 5 second timeout)
MongoDB server version: 3.6.0
<span class="ni">rs0:PRIMARY&gt; </span><span class="nc">show</span><span class="kv"> collections</span>
proxyGrantingTicketsCollection
proxyTicketsCollection
samlArtifactsCache
samlAttributeQueryCache
serviceTicketsCollection
ticketGrantingTicketsCollection
<span class="ni">rs0:PRIMARY&gt; </span>
</code></pre>
</div>

## Install on the CAS servers

Once everything is running correctly on the master build server, it can be copied to the CAS servers using the [updated build and installation scripts][building_svcmgmt_install-and-test-the-webapp]:

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# for host in srv01 srv02 srv03
> do
> scp -p /tmp/cassrv-files.tgz casdev-${host}:/tmp/cassrv-files.tgz
> scp -p /opt/scripts/cassrv-install.sh casdev-${host}:/tmp/cassrv-install.sh
> ssh casdev-${host} sh /tmp/cassrv-install.sh
> done
casdev-master#  
```

## Test the operation of the registry

To test the operation of the registry, start a terminal session on each of the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) and run the command

```console
# tail -f /var/log/cas/cas.log
```

Then:

1. Start a web browser and access the CAS client application (***casdev-casapp***). Click the link to access the secured content and log into CAS. Watch the terminal windows to see which CAS server processes the request. Note that it is quite likely (although not certain) that one server will handle the authnetication from your browser, and another server will handle the ticket validation from the client application.
2. Using the same web browser, access the SAML client application (***casdev-samlsp***). Again, click the link to access the secured content and watch the terminal windows to see which CAS server processes the request. There should be no need to re-enter your username and password, since the server should find your ticket granting ticket in the database.
3. Start a different web browser (or use a different computer) and repeat steps 1 and 2.
4. Start a different (from either of the first two) web browser, but this time access the status dashboard (`https://casdev.newschool.edu/cas/status/dashboard`). On the dashboard, click on the **SSO Sessions** button to see a list of all the current sessions (there should be one line per user that you have authenticated with, and the "Usage Count" column should show the number of services each user has authenticated to).
5. In the `mongo` shell, run the command

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">rs0:PRIMARY&gt; </span><span class="nc">db.ticketGrantingTicketsCollection.distinct(</span><span class="kv">"ticketId"</span><span class="nc">)</span>
[
        "TGT-1--H-sl9Yzq9a5gJYi1bsoV6ai_kukv7iD4_njJzuANUwZo6qosuX2r_3U-oxD5K0LBBg-casdev-srv01",
        "TGT-1-Blp3zphQTS-CS4JWf4Tb4u7c1Pl5i5TpK11f8-Eu5Sh-gAgjVi_KbRU1pqgQDYLLRL0-casdev-srv03",
        "TGT-1-KFfoL9mnLFMXx5il2Qyj-vMEN_3i5-i1dHBFoGuacr0COgybq-m5VK-E8k-ljXwoQMk-casdev-srv02"
]
<span class="ni">rs0:PRIMARY&gt; </span>
</code></pre>
</div>

to see that all of the tickets have been created in the database. Note that the end of each ticket identifier contains the host name of the particular CAS server that created it.

{% include reflinks.md %}
{% include links.html %}
