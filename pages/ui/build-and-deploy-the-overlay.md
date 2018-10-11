---
title: Build and deploy the overlay
last_updated: October 10, 2018
sidebar: main_sidebar
permalink: ui_build-and-deploy-the-overlay.html
summary:
---

Now that the theme and template files have been added to the overlay, the overlay has to be rebuilt and deployed. The customization process then becomes, basically:

1. Edit one or more of the files
2. Check the results in a web browser
3. Rinse and repeat

until happy. Doing this on the master build server and rebuilding/re-deploying the sever after every change is clearly not a very productive way to accomplish this process. Therefore, we will instead edit the files directly on the deployed CAS server, and once we're happy with them, copy the final versions back into the overlay on the master build server.

## Configure user interface properties

Edit the file `etc/cas/config/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and add the following line:

```properties
spring.thymeleaf.cache:                 false
```

This turns off caching in the Thymeleaf engine, so that when changes are made to the HTML views on the server, they will be immediately re-processed by Thymeleaf and visible when the browser reloads the pages. When caching is enabled, pages are only processed when the engine first loads them, meaning that changes will not be visible until the server is restarted.

{% include important.html content="Disabling Thymeleaf caching may have a negative impact on CAS server performance, especially in high-load environments. Therefore, it should only be disabled in development and/or test environments. On production CAS servers, this property should **always** be set to `true`." %}

Next, add the following line, which will make all services use the new theme and template files instead of the defaults:

```properties
cas.theme.defaultThemeName:             newschool
```

{% include note.html content="This setting forces all services to use the same theme, which makes sense in most environments. For environments that need (or want) to have different themes applied to different services, it's also possible to set the theme on a per-service basis via the service registry." %}

## Rebuild the server

Run Maven to rebuild the server to include the new theme and view files:

```console
casdev-master# ./mvnw clean package
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building cas-overlay 1.0
[INFO] ------------------------------------------------------------------------
(lots of diagnostic output... check for errors)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:00 min
[INFO] Finished at: YYYY-MM-DDTHH:MM:SS-00:00
[INFO] Final Memory: 35M/84M
[INFO] ------------------------------------------------------------------------
casdev-master#  
```

## Install and test on the master build server

Use the scripts [created earlier][building_server_install-and-test-the-cas-application] (or repeat the commands) to install the updated CAS configuration files on the master build server (***casdev-master***):

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# sh /opt/scripts/cassrv-install.sh
---Installing on casdev-master.newschool.edu
Installation complete.
casdev-master#  
```

Review the contents of the log files (`/var/log/tomcat/catalina.yyyy-mm-dd.out` and `/var/log/cas/cas.log`) for errors.

## Install on the CAS servers

Once everything is running correctly on the master build server, it can be copied to the CAS servers using the scripts [created earlier][building_server_install-and-test-the-cas-application]:

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# for host in srv01 srv02 srv03
> do
> scp -p /tmp/cassrv-files.tgz casdev-${host}:/tmp/cassrv-files.tgz
> scp -p /opt/scripts/cassrv-install.sh casdev-${host}:/tmp/cassrv-install.sh
> ssh casdev-${host} sh /tmp/cassrv-install.sh
> done
casdev-master#  
```

## Shut down all but one of the pool servers

Operating CAS with a pool of servers instead of a single server requires special configuration. Because that configuration hasn't been completed yet, testing must be performed against a single server. Further, since we're going to be editing the CSS, JavaScript, and HTML files and looking for those changes, we want to make sure we're accessing the same server (the one where we're editing the files) every time. Therefore, the other servers in the pool should be shut down so that the load balancer will direct all traffic to that single server. Run the command

```console
# systemctl stop tomcat
```

on all but one of the CAS servers (***casdev-srvXX***) to temporarily take those servers out of the pool.

## References

* [CAS 5: Configuration Properties: Themes][casdoc-config-themes]
* [CAS 5: Configuration Properties: Views][casdoc-config-views]

{% include reflinks.md %}
{% include links.html %}
