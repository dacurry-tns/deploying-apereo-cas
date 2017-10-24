---
title: Build the default server
last_updated: October 24, 2017
sidebar: main_sidebar
permalink: building_server_build-the-default-server.html
summary:
---

The Maven WAR overlay template will, out-of-the-box without any configuration, build a very basic "default" CAS server. This server doesn't do much, but it will let us verify that everything we've done up to this point is working correctly and give us a starting point for further configuration and customization. To build the default server, run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# ./mvnw clean package
Downloading https://repository.apache.org/content/repositories/releases/org/apache/maven/apache-maven/3.5.0/apache-maven-3.5.0-bin.zip
Unzipping /root/.m2/wrapper/dists/apache-maven-3.5.0-bin/766bhoj4b69i19aqdd66g707g1/apache-maven-3.5.0-bin.zip to /root/.m2/wrapper/dists/apache-maven-3.5.0-bin/766bhoj4b69i19aqdd66g707g1
Set executable permissions for: /root/.m2/wrapper/dists/apache-maven-3.5.0-bin/766bhoj4b69i19aqdd66g707g1/apache-maven-3.5.0/bin/mvn
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building cas-overlay 1.0
[INFO] ------------------------------------------------------------------------
(several hundred more lines of diagnostic output... check for errors)
```

on the master build server (***casdev-master***). Since this is our first time running `mvnw`, several hundred lines of diagnostic output will be printed as the wrapper downloads and installs Maven (to a cache directory), and as Maven downloads all the various software components that CAS servers are built from&mdash;CAS modules, Java libraries, third-party packages, etc.&mdash;from public software repositories and stores them in its cache. Once all that preparatory work is finished, the CAS application itself will be built:

```console
[INFO] Packaging webapp
[INFO] Assembling webapp [cas-overlay] in [/opt/workspace/cas-overlay-template/target/cas]
[info] Copying manifest...
[INFO] Processing war project
[INFO] Processing overlay [ id org.apereo.cas:cas-server-webapp-tomcat]
[INFO] Webapp assembled in [1086 msecs]
[INFO] Building war: /opt/workspace/cas-overlay-template/target/cas.war
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 02:10 min
[INFO] Finished at: YYYY-MM-DDTHH:MM:SS-00:00
[INFO] Final Memory: 13M/46M
[INFO] ------------------------------------------------------------------------
casdev-master#  
```

The end result of a successful build will be a subdirectory called `target` that contains a `cas.war` file:

```console
casdev-master# ls -asl target
total 77148
    0 drwxr-xr-x. 5 root root       61 Mmm dd hh:mm .
    4 drwxr-xr-x. 6 root root     4096 Mmm dd hh:mm ..
    0 drwxr-xr-x. 5 root root       45 Mmm dd hh:mm cas
89076 -rw-r--r--. 1 root root 91210098 Mmm dd hh:mm cas.war
    0 drwxr-xr-x. 2 root root       27 Mmm dd hh:mm maven-archiver
    0 drwxr-xr-x. 3 root root       17 Mmm dd hh:mm war
casdev-master#  
```

(ignore the other things in the `target` directory for now).
