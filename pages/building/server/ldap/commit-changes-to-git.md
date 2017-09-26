---
title: Commit changes to Git
last_updated: September 26, 2017
sidebar: main_sidebar
permalink: building_server_ldap_commit-changes-to-git.html
summary:
---

Before moving on to to the next phase of configuration, commit the changes made to `pom.xml` and `cas.properties`, as well as the new `etc/cas/services/ApacheSecuredByCAS-201700830155400.json` file, to Git to make them easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add etc/cas/config/cas.properties
casdev-master# git add etc/cas/services/ApacheSecuredByCAS-201700830155400.json
casdev-master# git add pom.xml
casdev-master# git commit
```

on the master build server (***casdev-master***). The `git commit` command will will bring up a text editor so you can describe the commit. Enter something like:

```
Add LDAP support:
 1. Add LDAP and SAML 1.1 modules to server
 2. Configure Active Directory authentication
 3. Configure Luminis LDAP authentication
 4. Configure AD/LDAP attribute resolution
 5. Create CasApp service definition
```

Then save and exit the editor, and Git will finish its work:

```console
[newschool-casdev 0cb7e85] Add LDAP support:  1. Add LDAP and SAML 1.1 modules to server  2. Configure Active Directory authentication  3. Configure Luminis LDAP authentication  4. Configure AD/LDAP attribute resolution  5. Create CasApp service definition
 3 files changed, 64 insertions(+)
 create mode 100644 etc/cas/services/ApacheSecuredByCAS-201700830155400.json
casdev-master#  
```
