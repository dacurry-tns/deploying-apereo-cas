---
title: Install Tomcat dependencies
last_updated: November 13, 2017
sidebar: main_sidebar
permalink: setup_tomcat_install-tomcat-dependencies.html
summary:
---

As discussed in the [introduction to this section][setup_tomcat_overview], the Tomcat Native Library is the best way to ensure that Tomcat meets TLS best practices. The library is included with Tomcat 8.5, but must be compiled and installed manually. To do that, the current versions of OpenSSL and the Apache Portable Runtime, which are not offered by Red Hat for RHEL 7, must also be downloaded, compiled, and installed. Lastly, we will also compile and install the Apache Commons Daemon to help with installing and running Tomcat as a system service.

All of the steps in this section should be performed on the master build server (***casdev-master***); the results will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) later.

## OpenSSL

The version of the Tomcat Native Library included with Tomcat 8.5 requires OpenSSL 1.0.2 or higher, which is not offered by Red Hat on RHEL7. Run the commands

```console
casdev-master# cd /tmp
casdev-master# curl -LO 'https://www.openssl.org/source/openssl-1.1.0g.tar.gz'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 5278k  100 5278k    0     0  33.4M      0 --:--:-- --:--:-- --:--:-- 33.4M
casdev-master# tar xzf openssl-1.1.0g.tar.gz
casdev-master# cd openssl-1.1.0g
casdev-master# mkdir -p /opt/openssl/openssl-1.1.0g
casdev-master# ln -s openssl-1.1.0g /opt/openssl/latest
casdev-master# ./config --prefix=/opt/openssl/openssl-1.1.0g shared
(lots of output... check for errors)
casdev-master# make depend
casdev-master# make
(lots of output... check for errors)
casdev-master# make test
(lots of output... check for errors)
casdev-master# make install_sw
(lots of output... check for errors)
casdev-master# cd /tmp
casdev-master# rm -rf openssl-1.1.0g openssl-1.1.0g.tar.gz
```

to download and build OpenSSL on the master build server (***casdev-master***). (Replace `1.1.0g` in the commands above with the current stable version of OpenSSL.)

{% include important.html content="The `INSTALL` document in the OpenSSL source directory says that tests **must** be run as an unprivileged user. If you decide to ignore this and run them as `root` anyway, you should expect the `40-test_rehash.t` test to fail." %}

This will install the current version of OpenSSL in a version-specific subdirectory of `/opt/openssl`, and make it accessible by the path `/opt/openssl/latest`. By linking the Tomcat Native Library against `/opt/openssl/latest`, an updated version of OpenSSL can be installed and the link changed without having to recompile anything else.

## Apache Portable Runtime

The Tomcat Native Library also depends on the Apache Portable Runtime (APR) library. Although the version of the APR library provided by Red Hat with RHEL7 is compatible with the Tomcat Native Library included with Tomcat 8.5, it's several versions behind the current release. Since we're building other dependencies anyway, we'll build this one too, just for completeness. Run the commands

```console
casdev-master# cd /tmp
casdev-master# curl -LO 'http://apache.cs.utah.edu//apr/apr-1.6.3.tar.gz'
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                               Dload  Upload   Total   Spent    Left  Speed
100 1045k  100 1045k    0     0   439k      0  0:00:02  0:00:02 --:--:--  439k
casdev-master# tar xzf apr-1.6.3.tar.gz
casdev-master# cd apr-1.6.3
casdev-master# mkdir -p /opt/apr/apr-1.6.3
casdev-master# ln -s apr-1.6.3 /opt/apr/latest
casdev-master# ./configure --prefix=/opt/apr/apr-1.6.3
casdev-master# make
(lots of output... check for errors)
casdev-master# make test
(lots of output... check for errors)
casdev-master# make install
(lots of output... check for errors)
casdev-master# cd /tmp
casdev-master# rm -rf apr-1.6.3 apr-1.6.3.tar.gz
```

to download and build the APR library on the master build server (***casdev-master***). (Replace `1.6.3` in the commands above with the current stable version of the APR library.)

This will install the current version of the APR library in a version-specific subdirectory of `/opt/apr`, and make it accessible by the path `/opt/apr/latest`. By linking the Tomcat Native Library against `/opt/apr/latest`, an updated version of the APR library can be installed and the link changed without having to recompile anything else.

## Tomcat Native Library

The Tomcat Native Library source code is included as part of the Tomcat distribution; it just needs to be extracted, compiled, and installed. Run the commands

```console
casdev-master# cd /opt/tomcat/apache-tomcat-8.5.23/bin
casdev-master# tar xzf tomcat-native.tar.gz
casdev-master# cd tomcat-native-*-src/native
casdev-master# ./configure \
  --with-java-home=/usr/lib/jvm/java-openjdk \
  --with-apr=/opt/apr/latest/bin/apr-1-config \
  --with-ssl=/opt/openssl/latest \
  --prefix=/opt/tomcat/apache-tomcat-8.5.23
(lots of output... check for errors)
casdev-master# make
(lots of output... check for errors)
casdev-master# make install
(lots of output... check for errors)
casdev-master# cd ../..
casdev-master# rm -rf tomcat-native-*-src
```

to build the Tomcat Native Library on the master build server (***casdev-master***). (Replace `8.5.23` in the commands above with the version of Tomcat installed earlier.)

This will install the Tomcat Native Library in the `lib` directory of its associated Tomcat installation.

## Apache Commons Daemon (`jsvc`)

The Apache Commons Daemon (`jsvc`) allows Tomcat to be started as `root` to perform some privileged operations (such as binding to ports below 1024) and then switch identity to run as a non-privileged user, which is better from a security perspective. The daemon is included as part of the Tomcat distribution; it just needs to be extracted, compiled, and installed. Run the commands

```console
casdev-master# cd /opt/tomcat/apache-tomcat-8.5.23/bin
casdev-master# tar xzf commons-daemon-native.tar.gz
casdev-master# cd commons-daemon-*-native-src/unix
casdev-master# ./configure --with-java=/usr/lib/jvm/java-openjdk
(lots of output... check for errors)
casdev-master# make
(lots of output... check for errors)
casdev-master# mv jsvc ../..
casdev-master# cd ../..
casdev-master# rm -rf commons-daemon-*-native-src
```

to build the Tomcat Native Library on the master build server (***casdev-master***). (Replace `8.5.23` in the commands above with the version of Tomcat installed earlier.)

This will install the `jsvc` program in the bin directory of its associated Tomcat installation.

{% include links.html %}
