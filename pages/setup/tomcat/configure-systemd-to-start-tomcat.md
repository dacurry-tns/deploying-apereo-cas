---
title: Configure <code>systemd</code> to start Tomcat
last_updated: November 3, 2017
sidebar: main_sidebar
permalink: setup_tomcat_configure-systemd-to-start-tomcat.html
summary:
---

RHEL 7 uses `systemd` (instead of `init`) to manage system resources. A *unit* is any resource that `systemd` knows how to operate on and manage.

The steps below should be performed on the master build server (***casdev-master***); the results will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) later.


## Define Tomcat as a service unit

Create the file `/etc/systemd/system/tomcat.service` with the following contents to define Tomcat as a service unit to `systemd`:

```shell
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking
PIDFile=/var/run/tomcat.pid
UMask=0007

# Tomcat variables
Environment='JAVA_HOME=/usr/lib/jvm/java-openjdk'
Environment='CATALINA_PID=/var/run/tomcat.pid'
Environment='CATALINA_HOME=/opt/tomcat/latest'
Environment='CATALINA_BASE=/opt/tomcat/latest'
Environment='CATALINA_OPTS=-Xms512M -Xmx2048M -XX:+UseParallelGC -server'

# Needed to make use of Tomcat Native Library
Environment='LD_LIBRARY_PATH=/opt/tomcat/latest/lib'

ExecStart=/opt/tomcat/latest/bin/jsvc \
            -Dcatalina.home=${CATALINA_HOME} \
            -Dcatalina.base=${CATALINA_BASE} \
            -Djava.awt.headless=true \
            -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager \
            -Djava.util.logging.config.file=${CATALINA_BASE}/conf/logging.properties \
            -cp ${CATALINA_HOME}/bin/commons-daemon.jar:${CATALINA_HOME}/bin/bootstrap.jar:${CATALINA_HOME}/bin/tomcat-juli.jar \
            -pidfile ${CATALINA_PID} \
            -java-home ${JAVA_HOME} \
            -user tomcat \
            $CATALINA_OPTS \
            org.apache.catalina.startup.Bootstrap

ExecStop=/opt/tomcat/latest/bin/jsvc \
            -pidfile ${CATALINA_PID} \
            -stop \
            org.apache.catalina.startup.Bootstrap

[Install]
WantedBy=multi-user.target
```

## Enable the Tomcat service unit

Run the commands

```console
casdev-master# restorecon /etc/systemd/system/tomcat.service
casdev-master# chmod 644 /etc/systemd/system/tomcat.service
```

to assign the correct SELinux context and file permissions to the `tomcat.service` file, and run the command

```console
casdev-master# systemctl enable tomcat.service
```

to enable the Tomcat service in `systemd`. This will cause `systemd` to start Tomcat at system boot time. Additionally, the following commands may now be used to manually start, stop, restart, and check the status of the Tomcat service:

```console
casdev-master# systemctl start tomcat
casdev-master# systemctl stop tomcat
casdev-master# systemctl restart tomcat
casdev-master# systemctl status tomcat
```
