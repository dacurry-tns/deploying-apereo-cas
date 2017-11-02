---
title: Commit changes to Git
last_updated: November 2, 2017
sidebar: main_sidebar
permalink: building_server_saml_commit-changes-to-git.html
summary:
---

Before moving on to building the SAML client, commit the changes made to `pom.xml`, as well as the new `etc/cas/saml` directory and the IdP-specific service definition, to Git to make them easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add etc/cas/saml
casdev-master# git add pom.xml
casdev-master# git add etc/cas/services/SAML2CallbackProfile-1509029745.json
casdev-master# git commit -m "Added SAML IdP support"
[newschool-casdev 3bf73e3] Added SAML IdP support
 6 files changed, 224 insertions(+)
 create mode 100644 etc/cas/saml/idp-encryption.crt
 create mode 100644 etc/cas/saml/idp-encryption.key
 create mode 100644 etc/cas/saml/idp-metadata.xml
 create mode 100644 etc/cas/saml/idp-signing.crt
 create mode 100644 etc/cas/saml/idp-signing.key
 create mode 100644 etc/cas/services/SAML2CallbackProfile-1509029745.json
casdev-master#  
```

on the master build server (***casdev-master***).
