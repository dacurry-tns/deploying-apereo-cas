---
title: Configure admin pages properties
last_updated: November 3, 2017
sidebar: main_sidebar
permalink: building_server_dashboard_configure-admin-pages-properties.html
summary:
---

There are two types of endpoints supported by the CAS server, those that can be viewed and managed via the dashboard only, and those that can also be viewed and managed with the Spring Boot Administration server. Since we will not be using the Spring Boot Administration server in our deployment, the distinction is somewhat unimportant, except that it introduces a few extra configuration settings.

## Enable all the endpoints

Edit the file `etc/cas/config/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and add the following lines to enable the endpoints:

```properties
cas.adminPagesSecurity.actuatorEndpointsEnabled:        true
cas.monitor.endpoints.enabled:          true
endpoints.enabled:                      true
```

This will enable both the CAS-style endpoints and the Spring Boot-style endpoints. The above settings apply globally to all the endpoints; it is also possible to enable and disable each endpoint individually; see the configuration property references below.

## Configure endpoint security

The top-level `/status` endpoint is always secured by an IP address pattern (regular expression). If no other security is configured, all the endpoints underneath `/status` are also secured by that pattern. The endpoints underneath `/status` may also be secured by the CAS server itself, just like any other CAS-enabled service, or they may secured with Spring Security, which supports basic authentication (master username and password), database-based authentication, LDAP-based authentication, and other methods. If neither CAS nor Spring Security are configured to secure the endpoints underneath `/status`, then their security will rely solely on the IP addrees pattern as well.

For our installation, we will use the IP address pattern to secure the `/status` endpoint, and the CAS server to secure the endpoints underneath it.

Edit the file `etc/cas/config/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and make the changes in the following sections.

### Configure the IP address pattern

Locate the line for the `cas.adminPagesSecurity.ip` property which, as distributed with the CAS Maven WAR overlay, allows access only from the local host:

```properties
cas.adminPagesSecurity.ip=127\.0\.0\.1
```

The value of this property is a regular expression that is matched against the IP address of the incoming request. If there is a match, access is granted; if there is no match, access is denied. Change the setting to allow access from whatever address(es) should have access. In our case, we will allow access from (a) the IT department office subnet (`192.168.50.0/24`), and (b) the internal interfaces of the F5 load balancers (`192.168.1.10`, `192.168.1.20`) (more on the reason for this later):

```properties
cas.adminPagesSecurity.ip:              ^192\.168\.(50\.[0-9]{1,3}|1\.[12]0)$
```

### Mark the endpoints "not sensitive"

In addition to enabling/disabling endpoints, CAS allows each endpoint to be marked "sensitive" or "not sensitive." The term "sensitive" is a little confusing in this context (it comes from the Spring Security world). Basically, if an endpoint is marked "sensitive" then Spring Security will be used to control access to it. If the endpoint is not marked "sensitive" then CAS will be used to secure it. Since we want to use the CAS server to secure all the endpoints, add the following properties:

```properties
cas.monitor.endpoints.sensitive:        false
endpoints.sensitive:                    false
```

### Configure the CAS server settings

To use the CAS server to secure the endpoints, the URL of the CAS server's login page and the name of the service to be accessed must be configured. In addition, to limit access to the endpoints to a specific set of "administrator" users, a separate user file should be provided that lists these users and their roles. To do this, add the following properties:

```properties
cas.adminPagesSecurity.loginUrl:        ${cas.server.prefix}/login
cas.adminPagesSecurity.service:         ${cas.server.prefix}/status/dashboard
cas.adminPagesSecurity.users:           file:/etc/cas/config/admusers.properties

cas.adminPagesSecurity.adminRoles[0]:   ROLE_ADMIN
```

## Create the `admusers.properties` file

Create a file called `etc/cas/config/admusers.properties` in the `cas-war-overlay` directory on the master build server (***casdev-master***) that looks like this:

```properties
# This file lists the users who are allowed access to the CAS /status/*
# endpoints ("adminpages").
#
# The syntax for each line is:
#
# username=password,grantedAuthority[,grantedAuthority][,enabled|disabled]
#
gnarls=passwordnotused,ROLE_ADMIN
```

Additional users can be listed, one per line, below the first one. Since the users are authenticating to the CAS server (against Active Directory or LDAP) the password field is not needed, and can be filled with any string (e.g., `passwordnotused`). Each user may have up to two `grantedAuthority` values assigned; for the user to be able to access the endpoints, one of these values must match one of the values assigned to the `cas.adminPagesSecurity.adminRoles` property (see above). This file is also used by the [service management webapp][building_svcmgmt_overview], so having two possible values allows users to be given access to the endpoints, the webapp, or both.

## References

* [CAS 5: Configuration Properties: CAS Endpoints][casdoc-config-cas-ep]
* [CAS 5: Configuration Properties: Spring Boot Endpoints][casdoc-config-sb-ep]

{% include reflinks.md %}
{% include links.html %}
