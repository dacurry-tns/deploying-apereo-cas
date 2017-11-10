---
title: Commit changes to Git
last_updated: November 10, 2017
sidebar: main_sidebar
permalink: building_svcmgmt_commit-changes-to-git.html
summary:
---

Before moving on to the next task, commit the new service registry definition file in the `cas-overlay-template` directory, as well as all the changes in the `cas-services-management-overlay` directory (`pom.xml`, `etc/cas/config/management.properties`, and `etc/cas/config/log4j2-management.properties`), to Git to make changes easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add etc/cas/services/CASServiceManagement-1510002272.json
casdev-master# git commit -m "Added service definition for the services management webapp"
[newschool-casdev 5581373] Added service definition for the services management webapp
 1 file changed, 8 insertions(+)
 create mode 100644 etc/cas/services/CASServiceManagement-1510002272.json
casdev-master# cd ../cas-services-management-overlay
casdev-master# git add pom.xml
casdev-master# git add etc/cas/config/management.properties
casdev-master# git add etc/cas/config/log4j2-management.xml
casdev-master# git commit -m "Enabled the services management webapp"
[newschool-casdev-sm 7688bb1] Enabled the services management webapp
 4 files changed, 42 insertions(+), 21 deletions(-)
 rewrite etc/cas/config/management.properties (99%)
 delete mode 100644 etc/cas/config/users.properties
casdev-master#  
```

on the master build server (***casdev-master***).
