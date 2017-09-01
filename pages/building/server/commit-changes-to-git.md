---
title: Commit changes to Git
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_commit-changes-to-git.html
summary:
---

Before moving on to the next phase of configuration, commit the changes made so far to `log4j2.xml` and `cas.properties` to Git to make them easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add etc/cas/config/cas.properties
casdev-master# git add etc/cas/config/log4j2.xml
casdev-master# git commit
```

on the master build server (***casdev-master***). The `git commit` command will bring up a text editor so you can describe the commit. Enter something like:

```
Basic server configuration:
 1. Set server host name and url information
 2. Configure TGC and webflow encryption
 3. Put log files into /var/log/cas
 4. Change log file rotation scheme
```

Then save and exit the editor, and Git will finish its work:

```console
[newschool-casdev 63e0694] asic server configuration:  1. Set server host name and url information  2. Configure TGC and webflow encryption  3. Put log files into /var/log/cas  4. Change log file rotation scheme
 2 files changed, 42 insertions(+), 17 deletions(-)
 rewrite etc/cas/config/cas.properties (75%)
casdev-master#  
```
