---
title: Commit changes to Git
last_updated: December 21, 2017
sidebar: main_sidebar
permalink: high-avail_service-registry_commit-changes-to-git.html
summary:
---

Before moving on to the next task, commit the changes made to `pom.xml` and `cas.properties` in the `cas-overlay-template` directory to Git to make changes easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add etc/cas/config/cas.properties
casdev-master# git add pom.xml
casdev-master# git commit -m "Added MongoDB service registry"
[newschool-casdev bfc6d29] Added MongoDB service registry
 2 files changed, 7 insertions(+), 3 deletions(-)
casdev-master#  
```

on the master build server (***casdev-master***). Then do the same with `pom.xml` and `management.properties` in the `cas-services-management-overlay` directory:

```console
casdev-master# cd /opt/workspace/cas-services-management-overlay
casdev-master# git add etc/cas/config/management.properties
casdev-master# git add pom.xml
casdev-master# git commit -m "Added MongoDB service registry"
[newschool-casdev-sm 7098c1f] Added MongoDB service registry
 2 files changed, 29 insertions(+), 4 deletions(-)
casdev-master#  
```
