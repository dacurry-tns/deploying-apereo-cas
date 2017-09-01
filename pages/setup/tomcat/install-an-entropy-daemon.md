---
title: Install an entropy daemon
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_install-an-entropy-daemon.html
summary:
---

{% include note.html content="This step is recommended if running CAS on virtual Linux servers. It is not necessary if running CAS on physical Linux servers or Windows servers of either type." %}

A common problem on virtual Linux servers is that the `/dev/random` device will run low on entropy, because most of the sources the kernel uses to build up the entropy pool are hardware-based, and therefore do not exist in a virtual environment. If there's not enough entropy available when Tomcat is started, it can often take two or three minutes or longer for the server to start. Once Tomcat has started and the CAS application has been loaded, entropy is still required to establish secure (HTTPS) connections with authenticating users' browsers and protected applications. A lack of available entropy will adversely affect the performance of the application by limiting the rate at which connections can be processed.

To improve the size of the entropy pool on Linux, it's possible to feed random data from an external source into `/dev/random`. One way to do this is the `haveged` daemon, which uses the HAVEGE (HArdware Volatile Entropy Gathering and Expansion) algorithm to harvest the indirect effects of hardware events on hidden processor state (caches, branch predictors, memory translation tables, etc) to generate random bytes with which to fill `/dev/random` whenever the supply of random bits falls below the low water mark of the device. We will use this approach to avoid entropy depletion on the CAS servers.

Red Hat does not offer `haveged` on RHEL 7, but it can be installed from the Fedora Project's Extra Packages for Enterprise Linux ([EPEL][epel-repo]) repository.

## Install the EPEL repository

Run the commands

```console
# cd /tmp
# curl -LO https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 14848  100 14848    0     0  27561      0 --:--:-- --:--:-- --:--:-- 27547
# yum -y install epel-release-latest-7.noarch.rpm
# rm -f epel-release-latest-7.noarch.rpm
```

on the master build server (***casdev-master***) and the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) to install the EPEL repository. It is not necessary to install the EPEL repository on the client application servers (***casdev-casapp*** and ***casdev-samlsp***).

## Install `haveged`

Run the commands

```console
# yum -y install haveged
# systemctl enable haveged
Created symlink from /etc/systemd/system/multi-user.target.wants/haveged.service to /usr/lib/systemd/system/haveged.service.
# systemctl start haveged
```

on the master build server (***casdev-master***) and the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) to install, enable, and start `haveged`. It is not necessary to install `haveged` on the client application servers (***casdev-casapp*** and ***casdev-samlsp***).

## References

* [haveged&mdash;a simple entropy daemon][havege-daemon]
* [HAVEGE project][havege-project]

{% include reflinks.md %}
