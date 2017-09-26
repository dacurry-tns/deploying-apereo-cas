---
title: Install and test the service registry
last_updated: September 26, 2017
sidebar: main_sidebar
permalink: building_server_service-registry_install-and-test-the-service-registry.html
summary:
---

Before the service registry can be used, the rebuilt CAS application and the updated configuration files must be installed and tested.

## Install and test on the master build server

Use the scripts [created earlier][building_server_install-and-test-the-cas-application] (or repeat the commands) to install the updated CAS application and configuration files on the master build server (***casdev-master***):

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# sh /opt/scripts/cassrv-install.sh
---Installing on casdev-master.newschool.edu
Installation complete.
casdev-master#  
```

{% include note.html content="You may want to edit `cassrv-install.sh` and change the line that reads `rm -rf etc/cas/config` (around line 10) to read `rm -rf etc/cas/config etc/cas/services` instead, to ensure that repeated installations do not leave any old service definitions lying around." %}

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

{% include reflinks.md %}
{% include links.html %}
