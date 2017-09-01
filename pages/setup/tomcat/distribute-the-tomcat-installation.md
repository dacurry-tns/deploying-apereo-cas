---
title: Distribute the Tomcat installation to the CAS servers
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_distribute-the-tomcat-installation.html
summary:
---

Once the Tomcat server has been successfully built, configured, and tested on the master build server (***casdev-master***) as described in the previous sections, the installation can be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***). It is not necessary (or desirable) to install Tomcat on ***casdev-casapp*** or ***casdev-samlsp***.

## Create a distribution `tar` file

To distribute all relevant files in the Tomcat installation to the CAS servers, we will assemble them all into a single `tar` archive that can be copied to each server and extracted. First, run the commands

```console
casdev-master# systemctl stop tomcat
casdev-master# cd /opt/tomcat/latest
casdev-master# rm -rf logs/* work/*
```

to shut down the Tomcat server and remove files that don't need to be included in the archive. Then run the commands

```console
casdev-master# cd /
casdev-master# tar czf /tmp/tomcat-files.tgz etc/tomcat opt/apr opt/openssl opt/tomcat var/cache/tomcat var/lib/tomcat var/cache/tomcat etc/firewalld/services/tomcat-https.xml etc/systemd/system/tomcat.service
```

to create the `tar` archive in `/tmp/tomcat-files.tgz`.

## Create an installation shell script

In addition to extracting the `tar` archive on each CAS server, commands must be executed to create the `tomcat` user and group, reset the correct user and group ownership of the installation, open firewall ports, and start the Tomcat service. To make it easier to run these commands on each server, they can be collected into a shell script (called, for example, `/opt/scripts/tomcat-install.sh`) like this:

```shell
#!/bin/sh

echo "--- Installing on `hostname`"

if [ -f /tmp/tomcat-files.tgz ]
then
    cd /
    tar xzf /tmp/tomcat-files.tgz

    groupadd -r tomcat
    useradd -r -d /opt/tomcat -g tomcat -s /sbin/nologin tomcat

    for dir in . conf webapps
    do
        cd /opt/tomcat/latest/$dir
        chown -R root.tomcat .
    done

    for dir in logs temp work
    do
        cd /opt/tomcat/latest/$dir
        chown -R tomcat.tomcat .
    done

    restorecon /etc/firewalld/services/tomcat-https.xml
    firewall-cmd --zone=public --add-service=tomcat-https --permanent
    firewall-cmd --reload

    restorecon /etc/systemd/system/tomcat.service
    systemctl enable tomcat.service
    systemctl start tomcat

    rm -f /tmp/tomcat-files.tgz /tmp/tomcat-install.sh
    echo "Installation complete."
else
    echo "Cannot find /tmp/tomcat-files.tgz; nothing installed."
fi

exit 0
```

This script will extract the contents of the tar archive to the right places and then run all the necessary commands to finish the installation and start Tomcat.

{% include note.html content="There is nothing special about the directory `/opt/scripts` (you can create it wherever you like, or not at all), but since we will be creating several little \"helper\" shell scripts throughout this deployment process, it makes sense to collect them all in a common location on the master build server." %}

## Copy files to each server and run the installation script

To complete the setup of each CAS server, the `tar` archive and installation shell script need to be copied to each server, and the installation script executed. This can be done manually, or with a shell loop as shown below:

```console
casdev-master# for host in srv01 srv02 srv03
> do
> scp -p /tmp/tomcat-files.tgz casdev-${host}:/tmp/tomcat-files.tgz
> scp -p /opt/scripts/tomcat-install.sh casdev-${host}:/tmp/tomcat-install.sh
> ssh casdev-${host} sh /tmp/tomcat-install.sh
> done
casdev-master#  
```

## Test Tomcat on each server

Open up a web browser and enter the URL of Tomcat's TLS port on each CAS server:

```
https://casdev-srv01.newschool.edu:8443
https://casdev-srv02.newschool.edu:8443
https://casdev-srv03.newschool.edu:8443
```

As in the previous test, expect the browser to complain about the TLS/SSL certificate because the host name of the server does not match the name in the certificate. Verify that the Tomcat "success" page appears on each server, just as it did for the development server.

{% include reflinks.md %}
{% include links.html %}
