---
title: Install Tomcat
last_updated: February 1, 2018
sidebar: main_sidebar
permalink: setup_tomcat_install-tomcat.html
summary:
---

{% include note.html content="The steps in this and the next several sections should only be performed on the build server (***casdev-master***). After everything has been built, configured, and tested, the installation will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***)." %}

As discussed in the [introduction to this section][setup_tomcat_overview], Red Hat does not offer Tomcat 8.5 on RHEL 7, so it must be downloaded and installed manually. Run the commands

```console
casdev-master# mkdir -p /opt/tomcat
casdev-master# cd /opt/tomcat
casdev-master# curl -LO 'http://apache.cs.utah.edu/tomcat/tomcat-8/v8.5.27/bin/apache-tomcat-8.5.27.tar.gz'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 9313k  100 9313k    0     0  4036k      0  0:00:02  0:00:02 --:--:-- 4038k
casdev-master# tar xzf apache-tomcat-8.5.27.tar.gz
casdev-master# ln -s apache-tomcat-8.5.27 latest
casdev-master# rm -f apache-tomcat-8.5.27.tar.gz
```

to download and install Tomcat on the master build server (***casdev-master***). (Replace `8.5.27` in the commands above with the current stable version of Tomcat 8.5.) This will install the current version of Tomcat 8.5 in a version-specific subdirectory of `/opt/tomcat`, and make it accessible by the path `/opt/tomcat/latest`.  By making everything outside this directory refer to `/opt/tomcat/latest`, an updated version can be installed and the link changed without having to edit/recompile anything else.

{% include links.html %}
