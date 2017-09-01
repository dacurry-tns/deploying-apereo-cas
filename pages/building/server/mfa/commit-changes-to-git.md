---
title: Commit changes to Git
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_mfa_commit-changes-to-git.html
summary:
---

Before moving on to the next task, commit the changes made to `pom.xml` and `cas.properties`, as well as the new `etc/cas/services/casapp-cas-duo.json` file, to Git to make them easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add etc/cas/config/cas.properties
casdev-master# git add etc/cas/services/casapp-cas-duo.json
casdev-master# git add pom.xml
casdev-master# git commit -m "Added Duo MFA support"
[newschool-casdev 7a51280] Added Duo MFA support
 3 files changed, 38 insertions(+)
 create mode 100644 etc/cas/services/casapp-cas-duo.json
casdev-master#  
```

on the master build server (***casdev-master***). The `git commit` command will not bring up a text editor as it did last time, since we provided the commit message on the command line.
