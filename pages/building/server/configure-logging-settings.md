---
title: Configure logging settings
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_configure-logging-settings.html
summary:
---

The Log4J configuration file included with the Maven WAR overlay template will attempt to write the CAS server log files (not the Tomcat log files) to the root of the CAS web application directory. However, since part of our [Tomcat hardening procedure][setup_tomcat_harden-the-installation] includes removing write permission to this directory for the `tomcat` user, this will not work (and it's not a very good place for them anyway). So, just as we moved Tomcat's log files to `/var/log/tomcat`, we will move the CAS server's log files to `/var/log/cas`.

Edit the file `etc/cas/config/log4j2.xml` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and find the line that defines the `cas.log.dir` property (around line 9) and change its value to `/var/log/cas`, like this:

```xml
<Property name="cas.log.dir" >/var/log/cas</Property>
```

Then create the `/var/log/cas` directory and set the ownership and permissions appropriately:

```console
casdev-master# mkdir /var/log/cas
casdev-master# chown tomcat.tomcat /var/log/cas
casdev-master# chmod 750 /var/log/cas
```

Don't forget to run the three commands above on the individual CAS servers as well.

## Adjust the log file rotation strategy (optional)

By default, the CAS log files will be rotated whenever their size reaches 10MB. On a busy server, this can result in numerous log files being created in a single day, making it more difficult to find particular events in the logs. To switch to a time-based rotation strategy in which the log files are rotated once a day, edit the
`etc/cas/config/log4j2.xml` file again, and make the following changes:

1. In the `RollingFile` configuration for `cas.log` (around line 17), change the variable part of the `filePattern` attribute from `%d{yyyy-MM-dd-HH}-%i.log` to `%d{yyyy-MM-dd}.log` (remove the hour and sequence number from the pattern).
2. Remove the `size="10MB"` attribute from the `SizeBasedTriggeringPolicy` element (around line 22).
3. Add the attributes `interval="1" modulate="true"` to the `TimeBasedTriggeringPolicy` element (around line 23).

The end result should look like this:

```xml
<RollingFile name="file" fileName="${sys:cas.log.dir}/cas.log" append="true"
             filePattern="${sys:cas.log.dir}/cas-%d{yyyy-MM-dd}.log">
    <PatternLayout pattern="%d %p [%c] - &lt;%m&gt;%n"/>
    <Policies>
        <OnStartupTriggeringPolicy />
        <SizeBasedTriggeringPolicy />
        <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
    </Policies>
</RollingFile>
```

Repeat the above changes for `cas_audit.log` (starting around line 26) and `perfStats.log` (starting around line 36).

## References

* [CAS 5: Logging][casdoc-logging]

{% include reflinks.md %}
{% include links.html %}
