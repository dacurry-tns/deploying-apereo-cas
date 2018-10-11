---
title: Commit changes to Git
last_updated: October 11, 2018
sidebar: main_sidebar
permalink: ui_commit-changes-to-git.html
summary:
---

Before moving on to the next stage of development, commit the new custom user interface files to Git to make changes easier to keep track of (and to enable reverting to earlier configurations easier). Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add src
casdev-master# git commit -m "Created New School-branded user interface"
[master (root-commit) e2cb175] Created New School-branded user interface
 137 files changed, 5243 insertions(+)
 create mode 100644 src/main/resources/custom_messages.properties
 create mode 100644 src/main/resources/newschool.properties
 create mode 100644 src/main/resources/static/themes/newschool/css/admin.css
 (lots of output...)
 create mode 100644 src/main/resources/templates/newschool/protocol/openid/casOpenIdServiceFailureView.html
 create mode 100644 src/main/resources/templates/newschool/protocol/openid/casOpenIdServiceSuccessView.html
 create mode 100644 src/main/resources/templates/newschool/protocol/openid/user.html
casdev-master#  
```

on the master build server (***casdev-master***).

{% include reflinks.md %}
{% include links.html %}
