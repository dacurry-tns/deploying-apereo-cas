---
title: Commit changes to Git
last_updated: December 14, 2017
sidebar: main_sidebar
permalink: high-avail_ticket-registry_commit-changes-to-git.html
summary:
---

Before moving on to the next task, commit the changes made to `pom.xml`, `cas.properties`, and `log4j2.xml` to Git to make changes easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add etc/cas/config/cas.properties
casdev-master# git add etc/cas/config/log4j2.xml
casdev-master# git add pom.xml
casdev-master# git commit -m "Added MongoDB ticket registry"
[newschool-casdev 57302ae] Added MongoDB ticket registry
 3 files changed, 33 insertions(+), 4 deletions(-)
casdev-master#  
```

on the master build server (***casdev-master***).
