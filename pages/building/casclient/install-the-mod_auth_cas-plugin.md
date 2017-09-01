---
title: Install the <code>mod_auth_cas</code> plugin
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_casclient_install-the-mod_auth_cas-plugin.html
summary:
---

The `mod_auth_cas` plugin allows an Apache web server to interact with a CAS server via the CAS protocol. Red Hat does not offer this plugin for installation via `yum` however, so it must be downloaded and built from source code. We will build the plugin on the master build server (***casdev-master***) where the compilers and other development tools have been installed, and then copy it to the client server (***casdev-casapp***) for installation and use.

## Install pre-requisites

The `mod_auth_cas` plugin build process depends on the presence of development libraries and header files from other packages. Run the commands

```console
casdev-master# yum -y install httpd-devel
casdev-master# yum -y install openssl-devel
casdev-master# yum -y install libcurl-devel
```

to install them.

## Clone the `mod_auth_cas` project

Use Git to clone the `mod_auth_cas` project from GitHub. Run the commands

```console
casdev-master# cd /opt/workspace
casdev-master# git clone https://github.com/apereo/mod_auth_cas.git
Cloning into 'mod_auth_cas'...
remote: Counting objects: 1766, done.
remote: Total 1766 (delta 0), reused 0 (delta 0), pack-reused 1766
Receiving objects: 100% (1766/1766), 1.47 MiB | 0 bytes/s, done.
Resolving deltas: 100% (1060/1060), done.
casdev-master#  
```

This will make a local copy of all files in the project and store them in a directory called `mod_auth_cas`. It will also record the information needed for Git to keep the local copy of the files synchronized with the copy stored on GitHub, so that corrections and updates made by the project team can be incorporated.

{% include tip.html content="As an alternative to using Git to clone a repository, GitHub allows the files in a repository to be downloaded in a Zip archive. However, this method does not include the metadata that Git needs to keep the local copy in sync with the master repository." %}

## Build the plugin

Run the commands

```console
casdev-master# cd /opt/workspace/mod_auth_cas
casdev-master# autoreconf -ivf
(lots of output... check for errors)
casdev-master# ./configure
(lots of output... check for errors)
casdev-master# make
(lots of output... check for errors)
casdev-master#  
```

to build the plugin.

## Install the plugin on the client server

An Apache HTTPD plugin is really just a dynamic shared library that can be loaded at runtime. Run the commands

```console
casdev-master# scp src/.libs/mod_auth_cas.so casdev-casapp:/etc/httpd/modules/mod_auth_cas.so
mod_auth_cas.so                               100%  241KB 240.7KB/s   00:00
casdev-master# ssh casdev-casapp "chown root.root /etc/httpd/modules/mod_auth_cas.so; chmod 755 /etc/httpd/modules/mod_auth_cas.so"
casdev-master#  
```

to copy the `mod_auth_cas` module to the appropriate location on the server where Apache HTTP is installed (***casdev-casapp***).

## References

* [GitHub repo for `mod_auth_cas`][mod_auth_cas]

{% include reflinks.md %}
