---
title: Install and test the final result
last_updated: October 11, 2018
sidebar: main_sidebar
permalink: ui_install-and-test-the-final-result.html
summary:
---

Once the custom theme files have been finalized (or at least reached a steady intermediate state), they can be copied back into the overlay and the CAS server can be rebuilt and deployed for more extensive testing.

## Copy the "live" files back into the overlay

To begin, the "live" copy of the custom theme files must be copied back into the overlay. Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay
casdev-master# cd src/resources
casdev-master# ssh casdev-srv01 "cd /var/lib/tomcat/cas/WEB-INF/classes; tar cf - newschool.properties custom_messages.properties static/themes/newschool templates/newschool" | tar xf -
casdev-master# chown -R root.root .
casdev-master# chmod -R og+rX .
```

on the master build server (***casdev-master***) to copy the "live" files from the server where they were being edited (***casdev-srv01*** in the example) back into the overlay.

## Rebuild the server

Run Maven to rebuild the server to include the new theme and view files:

```console
casdev-master# cd ../..
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

## Install and test on the master build server

Use the scripts [created earlier][building_server_install-and-test-the-cas-application] (or repeat the commands) to install the updated CAS configuration files on the master build server (***casdev-master***):

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# sh /opt/scripts/cassrv-install.sh
---Installing on casdev-master.newschool.edu
Installation complete.
casdev-master#  
```

Review the contents of the log files (`/var/log/tomcat/catalina.yyyy-mm-dd.out` and `/var/log/cas/cas.log`) for errors.

## Install on the CAS servers

Once everything is running correctly on the master build server, it can be copied to the CAS servers using the scripts [created earlier][building_server_install-and-test-the-cas-application]:

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

## Shut down all but one of the pool servers

Operating CAS with a pool of servers instead of a single server requires special configuration. Because that configuration hasn't been completed yet, testing must be performed against a single server. Further, since we're going to be editing the CSS, JavaScript, and HTML files and looking for those changes, we want to make sure we're accessing the same server (the one where we're editing the files) every time. Therefore, the other servers in the pool should be shut down so that the load balancer will direct all traffic to that single server. Run the command

```console
# systemctl stop tomcat
```

on all but one of the CAS servers (***casdev-srvXX***) to temporarily take those servers out of the pool.

{% include reflinks.md %}
{% include links.html %}
