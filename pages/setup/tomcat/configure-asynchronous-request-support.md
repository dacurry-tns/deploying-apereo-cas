---
title: Configure asynchronous request support
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_configure-asynchronous-request-support.html
summary:
---

CAS 5 requires the servlet container to support asynchronous requests. To enable asynchronous request support in Tomcat, edit the file `/opt/tomcat/latest/conf/web.xml`, locate the definition of the default web application servlet (around line 103), and add the `<async-supported>` directive:

```xml
<servlet>
    <servlet-name>default</servlet-name>
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    <init-param>
        <param-name>debug</param-name>
        <param-value>0</param-value>
    </init-param>
    <init-param>
        <param-name>listings</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
    <async-supported>true</async-supported>
</servlet>
```

Then locate the definition of the JSP compiler and execution servlet (around line 251) and make the same addition:

```xml
<servlet>
    <servlet-name>jsp</servlet-name>
    <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
    <init-param>
        <param-name>fork</param-name>
        <param-value>false</param-value>
    </init-param>
    <init-param>
        <param-name>xpoweredBy</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>3</load-on-startup>
    <async-supported>true</async-supported>
</servlet>
```

The steps above should be performed on the master build server (***casdev-master***); the results will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) later.

## References

* [CAS 5: Servlet Container Configuration][casdoc-srvlt-cont]

{% include reflinks.md %}
