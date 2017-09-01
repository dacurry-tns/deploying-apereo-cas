---
title: Open HTTP/HTTPS ports in the firewall
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_httpd-php_open-http-https-ports-in-the-firewall.html
summary:
---

To communicate with client systems, HTTPD needs to be able to communicate on TCP ports 80 (HTTP) and 443 (HTTPS). Run the commands

```console
# firewall-cmd --zone=public --add-service=http --permanent
success
# firewall-cmd --zone=public --add-service=https --permanent
success
# firewall-cmd --reload
success
#  
```

on ***casdev-casapp*** and ***casdev-samlsp*** to open these ports in the system firewall.
