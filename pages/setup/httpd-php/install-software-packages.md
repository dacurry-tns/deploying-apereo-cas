---
title: Install software packages
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_httpd-php_install-software-packages.html
summary:
---

As discussed in the [introduction to this section][setup_httpd-php_overview], we need to install Apache HTTPD and PHP on the servers. We also need to install the `mod_ssl` plugin for Apache, which enables TLS/SSL support. Run the commands

```console
# yum -y install httpd
# yum -y install mod_ssl
# yum -y install php
```

on ***casdev-casapp*** and ***casdev-samlsp*** to install these packages. It is not necessary (or desirable) to install HTTPD and PHP on ***casdev-srv01***, ***casdev-srv02***, or ***casdev-srv03***.

{% include links.html %}
