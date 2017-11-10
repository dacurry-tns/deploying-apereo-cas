---
title: Commit changes to Git
last_updated: November 10, 2017
sidebar: main_sidebar
permalink: building_server_dashboard_commit-changes-to-git.html
summary:
---

Before moving on to the next task, commit the changes made to `cas.properties`, as well as the new user file and new service registry definition file to Git to make changes easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# git add etc/cas/config/cas.properties
casdev-master# git add etc/cas/config/admusers.properties
casdev-master# git add etc/cas/services/CASAdminDaashboard-1509646291.json
casdev-master# git commit -m "Enabled the admin status dashboard"
[newschool-casdev 4e11f58] Enabled the admin status dashboard
 3 files changed, 52 insertions(+), 2 deletions(-)
 create mode 100644 etc/cas/config/admusers.properties
 create mode 100644 etc/cas/services/CASAdminDashboard-1509646291.json
casdev-master#  
```

on the master build server (***casdev-master***).
