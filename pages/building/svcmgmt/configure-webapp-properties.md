---
title: Configure webapp properties
last_updated: October 11, 2018
sidebar: main_sidebar
permalink: building_svcmgmt_configure-webapp-properties.html
summary:
---

Like the CAS server, the management webapp expects to find its configuration files in the operating system directory `/etc/cas`. The webapp configuration is controlled via settings in a separate properties file, `management.properties`, located in the `/etc/cas/config` directory. The Maven WAR overlay template provides a "source" for this file (which makes it easy to manage with Git).

Edit the file `etc/cas/config/management.properties` in the `cas-management-overlay` directory on the master build server (***casdev-master***) and make the changes in the following sections.

## Configure CAS server information

Locate the lines for the `cas.server.name` and `cas.server.prefix` properties (around lines 3-4) and *set them to the same values* as the identical properties in the `cas.properties` file:

```properties
cas.server.name:                       https://casdev.newschool.edu
cas.server.prefix:                     ${cas.server.name}/cas
```

These two properties tell the webapp to use the named CAS server for authentication.

## Configure webapp server name

Locate the line for the `cas.mgmt.serverName` property (around line 10) and set it to the URL of the server where the management webapp will be running. We're going to run the webapp on the CAS servers, in the same servlet container, so this property should have the same value as `cas.server.name`, above:

```properties
cas.mgmt.serverName:                   ${cas.server.name}
```

## Configure users and roles

Like the dashboard, the management webapp uses a separate users file to list the users who should be able to access it (after authenticating through the CAS server) and the role(s) they should have. Locate the line for the `cas.mgmt.userPropertiesFile` property (around line 7) and set it the same value as the `cas.adminPagesSecurity.users` property in the `cas.properties` file:

```properties
cas.mgmt.userPropertiesFile:           file:/etc/cas/config/admusers.properties
```

As discussed [previously][building_server_dashboard_configure-admin-pages-properties.html#create-the-admusersproperties-file], users in this file can have one or two roles assigned, making it possible to use different roles for the dashboard and the webapp. For simplicity, we will stick with the `ROLE_ADMIN` role, which is used out-of-the-box by both the dashboard and the webapp.

### Delete the `users.properties` file

Since we will be re-using the same users file that the dashboard is using, the users file included with the Maven WAR overlay is not needed. To avoid confusion, remove it from the project with the following `git` command:

```console
casdev-master# git rm etc/cas/config/users.properties
```

## Delete embedded servlet container properties

The `management.properties` file included with the Maven WAR overlay includes some properties that are only applicable when running the management webapp as a Spring Boot application in an embedded servlet container. Since we are using an external servlet container, these settings are not needed, and should be deleted. Locate  the lines below (around lines 12-13):

```properties
server.context-path=/cas-management
server.port=8443
```

Delete (or comment out) these lines.

## Configure the JSON service registry

In order for the management webapp to manage the services registry used by the CAS server, it has to know where it is. Add a new property, `cas.serviceRegistry.json.location`, and set it to the same value as the property of the same name in `cas.properties`:

```properties
cas.serviceRegistry.json.location:     file:/etc/cas/services
```

## References

* [CAS 5: Configuration Properties: Management Webapp][casdoc-config-svcmgmt-app]

{% include reflinks.md %}
{% include links.html %}
