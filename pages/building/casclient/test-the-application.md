---
title: Test the application
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_casclient_test-the-application.html
summary:
---

Now the use of CAS to protect the "secure" content created in the previous section can be tested by accessing the "public" part of the web site, and then clicking on the link to the "secure" section. At that point, the browser should be redirected to the CAS server, where a username and password can be entered. Provided that the username and password are correct, the secure content will be displayed.

Because both the load balancer and the CAS server use cookies, it's usually best to perform testing with an "incognito" or "private browsing" instance of the web browser that deletes all cookies each time it is closed.

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

{% include image.html file="building/casclient/fig07-the-public-site.png" alt="Browser Screen Shot" caption="Figure 7. The \"public\" site" %}

## Access the secure area

Click on the "here" link to access the secure content, and you will be redirected to the CAS server login page, as shown below:

{% include image.html file="building/casclient/fig08-the-cas-login-page.png" alt="Browser Screen Shot" caption="Figure 8. The CAS login page" %}

Note that the contents of the `name` field from the service registry are displayed in the middle of the right-hand column. Enter a valid username and password (`casuser` / `Mellon`) and, upon successful authentication, the contents of `/var/www/html/secured-by-cas/index.php` will be displayed:

{% include image.html file="building/casclient/fig09-the-secure-content.png" alt="Browser Screen Shot" caption="Figure 9. The \"secure\" content" %}

Note that the value shown for the `REMOTE_USER` variable is the username that was entered on the CAS login page (`casuser`).

## Restart the pool servers

One testing is complete, run the command

```console
# systemctl start tomcat
```

on each of the pool servers shut down previously.
