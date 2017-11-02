---
title: Commit changes to Git
last_updated: November 2, 2017
sidebar: main_sidebar
permalink: building_samlclient_commit-changes-to-git.html
summary:
---

Before moving on to building the SAML client, commit the new service registry definition file to Git to make changes easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# git add etc/cas/services/ApacheSecuredBySAML-1509030300.json
casdev-master# git commit -m "Set up SAML client"
[newschool-casdev 6ad660c] Set up SAML client
 1 file changed, 29 insertions(+)
 create mode 100644 etc/cas/services/ApacheSecuredBySAML-1509030300.json
casdev-master#  
```

on the master build server (***casdev-master***).
