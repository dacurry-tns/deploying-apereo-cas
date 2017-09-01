---
title: Organize the installation
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_organize-the-installation.html
summary:
---

To make it easier to upgrade Tomcat from one version to another without having to reapply configuration files changes or reinstall web applications, it makes sense to move these parts of the Tomcat installation to different parts of the file system and install symbolic links in their place.

All of the commands in this section should be run on the master build server (***casdev-master***); the results will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) later.

## Move the `conf` directory to `/etc/tomcat`

The `conf` subdirectory contains Tomcat's configuration files. Although these files can change from one major version to another, they don't typically have to change when installing newer minor versions. Run the commands

```console
casdev-master# cd /opt/tomcat/latest
casdev-master# cp -rp conf /etc/tomcat
casdev-master# rm -rf conf
casdev-master# ln -s /etc/tomcat conf
```

to move Tomcat's configuration files to `/etc/tomcat`.

## Move the `logs` directory to `/var/log/tomcat`

Log files can grow very large; storing them in `/opt` is not a good practice. Since log files are typically stored in `/var/log` on Linux systems, it makes sense to store Tomcat's log files there as well. Run the commands

```console
casdev-master# cd /opt/tomcat/latest
casdev-master# cp -rp logs /var/log/tomcat
casdev-master# rm -rf logs
casdev-master# ln -s /var/log/tomcat logs
```

to move Tomcat's log files to `/var/log/tomcat`.

## Move the `webapps` directory to `/var/lib/tomcat`

Although Tomcat runs web applications, the web applications themselves are not part of Tomcat. Therefore, it doesn't make a lot of sense to keep them inside the Tomcat installation directory, where they will have to be reinstalled every time Tomcat is updated. Run the commands

```console
casdev-master# cd /opt/tomcat/latest
casdev-master# cp -rp webapps /var/lib/tomcat
casdev-master# rm -rf webapps
casdev-master# ln -s /var/lib/tomcat webapps
```

to move the `webapps` directory to `/var/lib/tomcat`.

## Move the `work` directory to `/var/cache/tomcat/work`

Tomcat's `work` directory is where translated servlet source files and JSP/JSF classes are stored. Its contents are created automatically, but don't need to be recreated unless the application has been changed. To reduce startup time, the contents of this directory should be preserved across application restarts and system reboots. Linux systems provide the `/var/cache` directory for just that purpose, so we can put the `work` directory there. Run the commands

```console
casdev-master# cd /opt/tomcat/latest
casdev-master# mkdir /var/cache/tomcat
casdev-master# cp -rp work /var/cache/tomcat/work
casdev-master# rm -rf work
casdev-master# ln -s /var/cache/tomcat/work work
```

to move the `work` directory to `/var/cache/tomcat/work`. Note that we created a `work` subdirectory in `/var/cache/tomcat`; this is so that we can also use `/var/cache/tomcat` to store Tomcat's `temp` directory (see below).

## Move the `temp` directory to `/var/cache/tomcat/temp`

Tomcat provides a `temp` directory for web applications to store temporary files in. But like log files, temporary files can sometimes be very large, so storing them in `/opt` is probably not a good practice. But `/tmp` and `/var/tmp` are not the best places either, because we want to be able to limit access to Tomcat's temporary files (see [Harden the installation][setup_tomcat_harden-the-installation]). Therefore, we will create a new `temp` directory under `/var/cache/tomcat`. Run the commands

```console
casdev-master# cd /opt/tomcat/latest
casdev-master# cp -rp temp /var/cache/tomcat/temp
casdev-master# rm -rf temp
casdev-master# ln -s /var/cache/tomcat/temp temp
```

to move Tomcat's `temp` directory to `/var/cache/tomcat/temp`.

{% include links.html %}
