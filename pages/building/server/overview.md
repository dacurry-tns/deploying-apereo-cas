---
title: Building the CAS server
last_updated: October 11, 2018
sidebar: main_sidebar
permalink: building_server_overview.html
summary: Now that the development environment has been set up, CAS server development can begin with building and configuring a (very) basic server.
---

The Apache Maven build automation tool is used to configure and build the CAS server (CAS 4.2 and later also support using Gradle). Maven keeps track of the hundreds of library and object code dependencies associated with the CAS server and the particular features we have chosen to include, downloads the necessary files (in the appropriate versions) from public code repositories to a local cache, and assembles everything into a deployable bundle.

The CAS development team recommends that a WAR overlay project be used to organize feature selections and user interface design. This approach allows us to "overlay" our customizations&mdash;enabling or disabling features, setting configuration options, modifying the look and feel, etc.&mdash;onto a pre-built "vanilla" web application server provided by the CAS project itself, without having to download or build those components that we aren't using or changing.

We only have to manage the files that contain our customizations; Maven will take care of everything else.

## Create a work area

Because we will be working with more than one WAR overlay project (we will be creating separate ones later for the management application and the cloud configuration server), we'll create a top-level directory to keep them all in. Run the command

```console
casdev-master# mkdir /opt/workspace
```

to create a top-level directory on the master build server (***casdev-master***).

{% include note.html content="The directory may be created anywhere on the system; it does not have to reside under `/opt`. Furthermore, super-user permissions are not needed to build and configure the server (although they will be needed to deploy it)." %}

## References

* [CAS 5: WAR Overlay Installation][casdoc-war-overlay]
* [Apache Maven: Overlays][maven-overlays]

{% include reflinks.md %}
