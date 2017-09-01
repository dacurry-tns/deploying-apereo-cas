---
title: Install Java
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_install-java.html
summary:
---

CAS 5 requires Java 8 or higher. You can use either the OpenJDK or the Oracle version of the Java Development Kit (JDK), but if you opt for the Oracle version, you will also have to install the Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files to enable the use of stronger cryptographic algorithms and longer keys. Other than that, there is little practical difference between the two from CAS' perspective.

Since OpenJDK doesn't require us to download and maintain yet another package, we'll use that version for this deployment. Run the command

```console
# yum -y install java-1.8.0-openjdk-devel
```

on the master build server (***casdev-master***) and the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) to install OpenJDK Java 8. It is not necessary (or desirable) to install Java on the client application servers (***casdev-casapp*** and ***casdev-samlsp***).

Because it's possible to have more than one version of Java installed at the same time, run the command

```console
# java -version
openjdk version "1.8.0_141"
OpenJDK Runtime Environment (build 1.8.0_141-b16)
OpenJDK 64-Bit Server VM (build 25.141-b16, mixed mode)
#  
```

to check that Java 8 is indeed the default version (the version number should start with "1.8"; the remainder will vary depending on what the current patch level is).
