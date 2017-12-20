---
title: Harden the installation
last_updated: December 20, 2017
sidebar: main_sidebar
permalink: setup_tomcat_harden-the-installation.html
summary:
---

The *Tomcat Security Considerations* document makes several recommendations for hardening a Tomcat installation:

* Tomcat should not be run as the `root` user; it should be run as a dedicated user (usually named `tomcat`) that has minimum operating system permissions. It should not be possible to log in remotely as the `tomcat` user.

* All Tomcat files should be owned by user `root` and group `tomcat` (the `tomcat` user's default group should be group `tomcat`). File/directory permissions should be set to owner read/write, group read only, and world none. The exceptions are the `logs`, `temp`, and `work` directories, which should be owned by the `tomcat` user instead of `root`.

* The default and example web applications included with the Tomcat distribution should be removed if they are not needed.

* Auto-deployment should be disabled, and web applications should be deployed as exploded directories rather than web application archives (WAR files).

Implementing these recommendations means that, even if an attacker compromises the Tomcat process, he or she cannot change the Tomcat configuration, deploy new web applications, or modify existing web applications.

All of the steps in this section should be performed on the master build server (***casdev-master***); the results will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) later.

## Create a `tomcat` user and `tomcat` group

Run the commands

```console
casdev-master# groupadd -r tomcat
casdev-master# useradd -r -d /opt/tomcat -g tomcat -s /sbin/nologin tomcat
```

to create a `tomcat` user and a `tomcat` group.

## Set file ownership and permissions

Run the commands

```console
casdev-master# mkdir -p /opt/tomcat/latest/conf/Catalina/localhost
casdev-master# for dir in . conf webapps
> do
> cd /opt/tomcat/latest/$dir
> chown -R root.tomcat .
> chmod -R u+rwX,g+rX,o= .
> chmod -R g-w .
> done
casdev-master# for dir in logs temp work
> do
> cd /opt/tomcat/latest/$dir
> chown -R tomcat.tomcat .
> chmod -R u+rwX,g+rX,o= .
> chmod -R g-w .
> done
casdev-master#  
```

to set the proper file ownerships and permissions. Note that, as discussed above, some of the directories are owned by `root`, while others are owned by `tomcat`.

## Remove example webapps

Run the commands

```console
casdev-master# cd /opt/tomcat/latest
casdev-master# rm -rf temp/* work/*
casdev-master# cd webapps
casdev-master# rm -rf docs examples host-manager manager
```

to remove unneeded example web applications.

{% include important.html content="The command above does not remove the `ROOT` web application from the `webapps` directory because it can be useful in a development/test environment to quickly determine whether Tomcat is working properly. However, when deploying Tomcat to production servers, the `ROOT` application should be removed along with the rest of the default web applications." %}

## Disable auto-deployment

To disable auto-deployment, edit the file `/opt/tomcat/latest/conf/server.xml` and locate the `Host` XML tag (around line 148), which should look something like this:

```xml
<Host name="localhost"  appBase="webapps"
      unpackWARs="true" autoDeploy="true">
```

Disable application auto-deployment and unpacking of web application archive files by setting these attributes to `false`:

```xml
<Host name="localhost"  appBase="webapps"
      unpackWARs="false" autoDeploy="false">
```

## References

* [Tomcat Security Considerations][tomcat-sec-con]

{% include reflinks.md %}
