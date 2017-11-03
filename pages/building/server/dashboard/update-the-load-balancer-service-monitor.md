---
title: Update the load balancer service monitor
last_updated: November 3, 2017
sidebar: main_sidebar
permalink: building_server_dashboard_update-the-load-balancer-service-monitor.html
summary:
---

When we first [configured the load balancers][setup_tomcat_configure-the-load-balancers], we defined a monitor for the server pool that connected to each server every 5 seconds and issued a `GET /` HTTP request to verify that Tomcat was running. Later, when we [built and installed the CAS server][building_server_install-and-test-the-cas-application], we adjusted this monitor to instead issue a `GET /cas/login` request and check for some text on the login page, to verify that not only Tomcat, but the CAS application itself, were running. The drawback to this change is that it resulted in a huge number of started-but-never-finished login sessions on the CAS servers, and a correspondingly huge amount of output in the log files (which, in addition to wasting disk space, made the logs much harder to read).

However, now that we have the `/status` endpoint configured, we can adjust the monitor to issue a `GET /cas/status` request instead. The output from a `GET /cas/status` request looks something like this:

```
Health: OK

	1.SessionMonitor: OK - 2 sessions. 0 service tickets.

	2.MemoryMonitor: OK - 1648.57MB free (90.55%), 172.04MB used, 1820.61MB total.

Host:		casdev-srv01
Server:		https://casdev.newschool.edu
Version:	5.2.0-SNAPSHOT
```

So, by checking the data returned from the server for `Health: OK`, we can still verify that Tomcat and the CAS server are running. To do this, log into the F5 administration portal and edit the `casdev_https_8443_monitor`. Change these two lines in the definition:

```tcl
recv "Login - CAS"
send "GET /cas/login\\r\\n"
```

to:

```tcl
recv "Health: OK"
send "GET /cas/status\\r\\n"
```

so that the entire monitor looks like this:

```tcl
ltm monitor https /Common/casdev_https_8443_monitor {
    adaptive disabled
    cipherlist DEFAULT:+SHA:+3DES:+kEDH
    compatibility enabled
    defaults-from /Common/https
    description "Cas Dev Application HTTPS Monitor"
    destination *:8443
    interval 5
    is-dscp 0
    recv "Health: OK"
    recv-disable none
    send "GET /cas/status\\r\\n"
    time-until-up 0
    timeout 16
}
```

With the new monitor, there will still be a single line written to the Tomcat access log, but there will not be any more lines written to the CAS log file.

{% include reflinks.md %}
{% include links.html %}
