---
title: Tune resource caching settings
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_tune-resource-caching-settings.html
summary:
---

To improve performance, Tomcat is configured by default to cache static resources. However, the size of the cache is too small to work effectively with the CAS application. To tune Tomcat's cache settings, edit the file `/opt/tomcat/latest/conf/context.xml`, locate the definition of the default context (around line 19), and add a `<Resources>` directive at the bottom:

```xml
<Context>
    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
    <!--
    <Manager pathname="" />
    -->

    <!-- Enable caching, increase the cache size (10240 default), increase   -->
    <!-- the ttl (5s default)                                                -->
    <Resources cachingAllowed="true" cacheMaxSize="40960" cacheTtl="60000" />
</Context>
```

The step above should be performed on the master build server (***casdev-master***); the results will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) later.
