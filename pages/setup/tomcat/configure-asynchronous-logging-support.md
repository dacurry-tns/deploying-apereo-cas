---
title: Configure asynchronous logging support
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_configure-asynchronous-logging-support.html
summary:
---

CAS 5's logging subsystem automatically inserts itself into the runtime application context at startup time, and is designed to clean up the logging context when Tomcat shuts down. Unfortunately, the default Tomcat JarScanner configuration skips over JAR files named `log4j*.jar`, which prevents this feature from working.

To correct this problem, edit the file `/opt/tomcat/latest/conf/catalina.properties` and locate the lines defining the `jarsToSkip` property (around lines 108-134), and then the specific line of that definition that includes `log4j*.jar` (around line 128):

```
tomcat.util.scan.StandardJarScanFilter.jarsToSkip=\
...
jmx-tools.jar,jta*.jar,log4j*.jar,mail*.jar,slf4j*.jar,\
...
```

and remove `log4j*.jar` from that line:

```
tomcat.util.scan.StandardJarScanFilter.jarsToSkip=\
...
jmx-tools.jar,jta*.jar,mail*.jar,slf4j*.jar,\
...
```

The step above should be performed on the master build server (***casdev-master***); the results will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) later.

## References

* [CAS 5: Servlet Container Configuration][casdoc-srvlt-cont]
* [Tomcat Configuration Reference: The Jar Scanner Component][tomcat-jarscanner]

{% include reflinks.md %}
