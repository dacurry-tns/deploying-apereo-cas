---
title: Open TLS/SSL port in the firewall
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_open-tlsssl-port-in-the-firewall.html
summary:
---

For client systems to be able to communicate with the CAS server, the TCP port that Tomcat's HTTPS connector was configured to use earlier (see [Enable and configure the HTTPS connector][setup_tomcat_configure-tlsssl-settings.html#enablehttpsconnector]) must be opened in the operating system firewall. To do this, first create a `firewalld` service configuration file called `/etc/firewalld/services/tomcat-https.xml` with the following contents:

```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Tomcat Secure HTTP (HTTPS)</short>
  <description>Tomcat typically implements TLS/SSL-secured HTTP (HTTPS) on a different port than a regular web server does (often so that both servers can co-exist on the same system).</description>
  <port protocol="tcp" port="8443"/>
</service>
```

to define the Tomcat HTTPS service. Then, run the commands

```console
casdev-master# restorecon /etc/firewalld/services/tomcat-https.xml
casdev-master# chmod 640 /etc/firewalld/services/tomcat-https.xml
```

to assign the correct SELinux context and file permissions to the `tomcat-https.xml` file. Finally, run the commands

```console
casdev-master# firewall-cmd --zone=public --add-service=tomcat-https --permanent
success
casdev-master# firewall-cmd --reload
success
casdev-master#  
```

to open the newly-defined service in the system firewall.

The steps above should be performed on the master build server (***casdev-master***); the results will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) later.

{% include links.html %}
