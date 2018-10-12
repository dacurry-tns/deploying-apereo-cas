---
title: Install and test the application
last_updated: October 12, 2018
sidebar: main_sidebar
permalink: googleapps_install-and-test-the-application.html
summary:
---

Google Apps single sign-on can be tested by installing the updated CAS software on the CAS servers and restarting it, and then signing into Google.

## Install and test on the master build server

Use the [updated build and installation scripts][building_svcmgmt_install-and-test-the-webapp] (or repeat the commands) to install the updated CAS server on the master build server (***casdev-master***) and restart Tomcat:

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# sh /opt/scripts/cassrv-install.sh
---Installing on casdev-master.newschool.edu
Installation complete.
casdev-master#  
```

Review the contents of the log files (`/var/log/tomcat/catalina.yyyy-mm-dd.out` and `/var/log/cas/cas.log`) for errors.

## Install on the CAS servers

Once everything is running correctly on the master build server, it can be copied to the CAS servers using the [updated build and installation scripts][building_svcmgmt_install-and-test-the-webapp]:

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

## Register Google Apps in the service registry

Using the management webapp, add a new service registry entry for the Google Apps instance. The service URL should be provided as:

```
^https://www.google.com/a/[Google Apps domain]/acs(\z|/.*)
```

For The New School then, this would be:

```
^https://www.google.com/a/newschool.edu/acs(\z|/.*)
```

{% include important.html content="Although it uses the SAML2 protocol, Google Apps should be registered as a ***CAS client*** service, not a SAML2 Service Provider. That is, it should be registered as a `org.apereo.cas.services.RegexRegisteredService`, not as a `org.apereo.cas.support.saml.services.SamlRegisteredService`." %}

If the Google Apps instance does not use the same usernames as the CAS server, the service registry entry can be configured to return the alternate username on the *Username Attribute* tab of the management webapp.

## Test the operation of Google Apps

To test the operation of Google Apps, visit `https://mail.google.com/a/[Google Apps domain]` (for The New School, `https://mail.google.com/a/newschool.edu`).

## References

* [CAS 5: Google Apps Integration][casdoc-googleapps]

{% include reflinks.md %}
{% include links.html %}
