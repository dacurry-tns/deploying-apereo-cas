---
title: Configure <code>systemd</code> to start HTTPD
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_httpd-php_configure-systemd-to-start-httpd.html
summary:
---

RHEL 7 uses `systemd` (instead of `init`) to manage system resources. Run the command

```console
# systemctl enable httpd.service
```

on ***casdev-casapp*** and ***casdev-samlsp*** to enable the HTTPD service in `systemd`. This will cause `systemd` to start HTTPD at system boot time. Additionally, the following commands may now be used to manually start, stop, restart, and check the status of the HTTPD service:

```console
# systemctl start httpd
# systemctl stop httpd
# systemctl restart httpd
# systemctl status httpd
```
