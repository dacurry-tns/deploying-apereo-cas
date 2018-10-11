---
title: Overview
last_updated: October 11, 2018
sidebar: main_sidebar
permalink: ui_develop_overview.html
summary:
---

Now that the files for the new theme have been put into the Maven overlay and deployed to the CAS servers, it's time to customize their content. This process can be broken down into four main steps:

1. Identify the "applies to all pages" elements of the HTML in the mock-up login page created earlier, and merge them into the layout template (`layout.html`).
2. Update the login view to use the new layout template, and copy the login page-specific elements of the HTML in the mock-up login page to the files that make up the login view (`casLoginView.html` and `fragments/loginform.html`).
3. Update the logout view to use the new layout template.
4. Update the other views used by the server to use the new layout template.

As mentioned previously, it's easiest to perform this work on the "live" files on one of the deployed CAS servers to avoid the need to rebuild and re-deploy the overlay after every change. Once an acceptable set of files has been created, they can be copied back to the source (overlay template) directory and committed to `git`. For the purposes of the following changes, ***casdev-srv01*** will be that server. Begin by changing to the deployed CAS server directory:

```console
casdev-srv01# cd /var/lib/tomcat/cas/WEB-INF/classes
```
