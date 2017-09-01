---
title: Configure <code>X-Forwarded-For</code> header processing
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_configure-xforwardedfor-header-processing.html
summary:
---

When Tomcat is installed behind a load balancer as it will be in our environment, incoming network connections will have a source address of the load balancer's internal interface rather than the address of the client system on the other side of the load balancer. This will have a negative impact on Tomcat (as well as CAS) logging, since the logs will not be able to identify individual systems by their network addresses. It will also prevent certain CAS features, such as adaptive authentication based on client address geolocation, from working correctly.

To correct this situation, most load balancers can be configured to insert an `X-Forwarded-For` HTTP header into the data stream to identify the address of the connecting client system, and Tomcat can be configured to look for this header and substitute the address provided for the source address attached to the connection from the load balancer.

To configure Tomcat to process `X-Forwarded-For` HTTP headers, edit the file `/opt/tomcat/latest/conf/server.xml` again and locate the definition of the `AccessLogValve` (around line 160, after inserting the changes in [Configure TLS/SSL settings][setup_tomcat_configure-tlsssl-settings]) and

1. Insert a `RemoteIpValve` definition above it.
2. Add a `requestAttributesEnabled` attribute to the `AccessLogValve` definition.

When finished, things should look something like this:

```xml
<!-- RemoteIp valve, process X-Forwarded-For headers
     Documentation at: /docs/config/valve.html -->
<Valve className="org.apache.catalina.valves.RemoteIpValve"
       internalProxies="192\.168\.1\.10" />

<!-- Access log processes all example.
     Documentation at: /docs/config/valve.html
     Note: The pattern used is equivalent to using pattern="common" -->
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
       prefix="localhost_access_log" suffix=".txt"
       requestAttributesEnabled="true"
       pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```

The value of the `internalProxies` attribute on the `RemoteIpValve` declaration should be a regular expression that matches the IP address(es) of the internal interface(s) of the load balancer(s). Since '.' is a special character in regular expressions, it should be escaped with a backslash. To represent multiple IP addresses, separate them with '\|' symbols (for example, `192\.168\.1\.10|192\.168\.1\.20`).

The steps above should be performed on the master build server (***casdev-master***); the results will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) later.

## References

* [Tomcat Configuration Reference: The Valve Component][tomcat-valve]

{% include reflinks.md %}
{% include links.html %}
