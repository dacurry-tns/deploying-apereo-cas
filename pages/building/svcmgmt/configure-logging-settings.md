---
title: Configure logging settings
last_updated: November 8, 2017
sidebar: main_sidebar
permalink: building_svcmgmt_configure-logging-settings.html
summary:
---

The services management webapp includes its own Log4j configuration. [As we did with the CAS server][building_server_configure-logging-settings], we will move the location of the log file from the root of the web application directory to `/var/log/cas`.

Edit the file `etc/cas/config/log4j2-management.xml` in the `cas-services-management-overlay` directory on the master build server (***casdev-master***) and find the line that defines the `cas.log.dir` property (around line 9) and change its value to `/var/log/cas`, like this:

```xml
<Property name="cas.log.dir" >/var/log/cas</Property>
```

## Adjust the log file rotation strategy (optional)

By default, the webapp log file will be rotated whenever its size reaches 512KB. To switch to same time-based rotation strategy we established for the CAS server, edit the
`etc/cas/config/log4j2-management.xml` file again, and make the following changes:

1. In the `RollingFile` configuration for `cas.log` (around line 17), change the variable part of the `filePattern` attribute from `%d{yyyy-MM-dd-HH}-%i.log` to `%d{yyyy-MM-dd}.log` (remove the hour and sequence number from the pattern).
2. Remove the `size="512 KB"` attribute from the `SizeBasedTriggeringPolicy` element (around line 22).
3. Add the attributes `interval="1" modulate="true"` to the `TimeBasedTriggeringPolicy` element (around line 23).

The end result should look like this:

```xml
<RollingFile name="cas-management" fileName="${sys:cas.log.dir}/cas-management.log" append="true"
             filePattern="${sys:cas.log.dir}/cas-management-%d{yyyy-MM-dd}.log">
    <PatternLayout pattern="%d %p [%c] - %m%n"/>
    <Policies>
        <OnStartupTriggeringPolicy />
        <SizeBasedTriggeringPolicy />
        <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
    </Policies>
</RollingFile>
```

## References

* [CAS 5: Logging][casdoc-logging]

{% include reflinks.md %}
{% include links.html %}
