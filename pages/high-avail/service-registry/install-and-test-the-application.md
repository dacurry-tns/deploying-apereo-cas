---
title: Install and test the application
last_updated: October 11, 2018
sidebar: main_sidebar
permalink: high-avail_service-registry_install-and-test-the-application.html
summary:
---

The MongoDB service registry can be tested by installing the updated CAS software on the CAS servers and restarting it, and then accessing the test clients and the management webapp.

## Delete the JSON service registry

Before installing, delete the JSON service registry on the master build server and the CAS servers by either renaming or removing the `/etc/cas/services` directory:

```console
casdev-master# mv /etc/cas/services /etc/cas/services.off
casdev-master# for i in 01 02 03
> do
> ssh casdev-srv${i} rm -rf /etc/cas/services
> done
casdev-master#  
```

## Install and test on the master build server

Use the [updated build and installation scripts][building_svcmgmt_install-and-test-the-webapp] (or repeat the commands) to install the updated CAS server on the master build server (***casdev-master***) and restart Tomcat:

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# sh /opt/scripts/cassrv-install.sh
---Installing on casdev-master.newschool.edu
Installation complete.
casdev-master#  
```

Review the contents of the log files (`/var/log/tomcat/catalina.yyyy-mm-dd.out` and `/var/log/cas/cas.log`) for errors.

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

To test the operation of the registry, log into the management webapp and verify that all expected services are present, and that a service can be added to the registry (and removed from it). You can also use the CAS and SAML client applications to ensure that you can access the protected areas of those applications and that all expected attributes are still being released.

{% include reflinks.md %}
{% include links.html %}
