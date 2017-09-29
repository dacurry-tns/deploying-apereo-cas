---
title: Install and test the application
last_updated: September 29, 2017
sidebar: main_sidebar
permalink: building_server_mfa_install-and-test-the-application.html
summary:
---

Before multifactor authentication can be used, the rebuilt CAS application and the updated configuration files must be installed. Then everything can be tested by accessing the "public" part of the client application web site and then clicking on the link to the "secure" section. At that point, the browser should be redirected to the CAS server, where upon successful authentication the secure content, including the values of the attributes, will be displayed.

Because both the load balancer and the CAS server use cookies, it's usually best to perform testing with an "incognito" or "private browsing" instance of the web browser that deletes all cookies each time it is closed.

## Install and test on the master build server

Use the scripts [created earlier][building_server_install-and-test-the-cas-application] (or repeat the commands) to install the updated CAS application and configuration files on the master build server (***casdev-master***):

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

Operating CAS with a pool of servers instead of a single server requires special configuration. Because that configuration hasn't been completed yet, testing must be performed against a single server. Therefore, the other servers in the pool should be shut down so that the load balancer will direct all traffic to that single server. Run the command

```console
# systemctl stop tomcat
```

on all but one of the CAS servers (***casdev-srvXX***) to temporarily take those servers out of the pool.

## Access the public site

Open up a web browser (in "incognito" or "private browsing" mode) and enter the URL of the CAS-enabled web site:

```
https://casdev-casapp.newschool.edu/
```

The contents of `/var/www/html/index.php` should be displayed, looking something like this:

{% include image.html file="building/server/mfa/fig14-the-public-site.png" alt="Browser Screen Shot" caption="Figure 14. The \"public\" site" %}

## Access the secure area

Click on the second "here" link to access the content secured by CAS and Duo, and you will be redirected to the CAS server login page, as shown below:

{% include image.html file="building/server/mfa/fig15-the-cas-login-page.png" alt="Browser Screen Shot" caption="Figure 15. The CAS login page" %}

Note that the contents of the `name` field from the service registry are displayed at the top of the right-hand column; make sure that `Apache Secured by CAS and Duo` is displayed here. Enter a valid username and password (Active Directory or LDAP) that is also registered with Duo and, upon successful first-stage authentication, the Duo MFA authentication page will appear, as shown below:

{% include image.html file="building/server/mfa/fig16-the-duo-authentication-page.png" alt="Browser Screen Shot" caption="Figure 16. The Duo authentication page" %}

Upon successful Duo authentication, the contents of `/var/www/html/secured-by-cas-duo/index.php` will be displayed:

{% include image.html file="building/server/mfa/fig17-the-secure-content.png" alt="Browser Screen Shot" caption="Figure 17. The \"secure\" content" %}

## Restart the pool servers

One testing is complete, run the command

```console
# systemctl start tomcat
```

on each of the pool servers shut down previously.

{% include links.html %}
