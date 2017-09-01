---
title: Initial setup tasks
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_initial-setup-tasks.html
summary:
---

Before starting the process of configuring the various servers to perform their individual roles in the development environment, there are some initial setup tasks to be performed.

## Ensure that all systems are up-to-date

It's important to make sure that the operating system software on the servers is up-to-date. Run the command

```console
# yum -y update
```
on each of the six servers in the environment to ensure that all software installed on the base system is up-to-date. If running this command results in updates to system shared libraries or the operating system kernel, reboot the server(s) before continuing.

## Install development tools on the master build server

The master build server (***casdev-master***) will be used to build and compile software from source code. Run the command

```console
casdev-master# yum -y groupinstall "Development Tools"
```

to install the tools needed to do that. This command should only be run on the master build server (***casdev-master***); it is not necessary (or desirable) to install these tools on any of the other servers.

## Install Perl test modules on the master build server

Some of the software packages to be installed use Perl testing modules to perform their tests. Run the commands

```console
casdev-master# yum -y install perl-Module-Load-Conditional
casdev-master# yum -y install perl-Test-Simple
```

to install the necessary modules. This command should only be run on the master build server (***casdev-master***); it is not necessary to install these modules on any of the other servers.

## Configure Git (optional)

The CAS project team uses GitHub to host all of its code, maintain version control, and allow collaboration among developers. Once we start [Building the CAS server][building_server_overview], we'll be using Git commands to make local copies of files from the CAS GitHub repositories, to track our changes and additions to those files as we customize them, and to keep our local files in sync with any corrections and updates made to the master copies by the project team.

{% include tip.html content="If you're unfamiliar with Git and GitHub, you may want to read [*Pro Git*][pro-git] (chapters 2, 3, and 6)." %}

When making a Git commit (recording a change to a file as a new version), Git requires that the user name and email address of the person making the commit be provided so they can be recorded in the commit history. To avoid having to specify these values every time, they can be configured ahead of time by running the commands

```console
casdev-master# git config --global user.name "David A. Curry"
casdev-master# git config --global user.email "david.curry@newschool.edu"
```

on the master build server (***casdev-master***). (Substitute your name and email address for the values inside the quotation marks.) When making a commit, Git will also ask for some text to describe what was changed, and will invoke a text editor to allow that text to be entered. Run the command

```console
casdev-master# git config --global core.editor "vim"
```

to configure the editor that will be used for this purpose (the example above sets the editor to `vim`; another common choice is `emacs`).

## Set up SSH public key authentication (optional)

To make it easier to distribute the software built on ***casdev-master*** to the other servers in the environment, we will create a public/private authentication key pair that will allow ***casdev-master*** to connect to those servers via `ssh` and `scp` without a password. Run the command

```console
casdev-master# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
89:e0:9e:11:78:64:64:22:d3:df:74:30:38:e3:ec:c6 root@casdev-master.newschool.edu
The key's randomart image is:
+--[ RSA 2048]----+
|o...=.o.         |
| o.*+ ...        |
|  .++= .         |
|   o+o.. .       |
|   oo . S        |
|   .Eo           |
|   .o            |
|                 |
|                 |
+-----------------+
casdev-master#  
```

on the master build server (***casdev-master***) to generate the key pair. Once the key pair has been generated, run the command

```console
casdev-master# ssh-copy-id casdev-srv01.newschool.edu
The authenticity of host 'casdev-srv01.newschool.edu (192.168.20.1)' can't be established.
ECDSA key fingerprint is 43:51:43:a1:b5:fc:8b:b7:0a:3a:a9:b1:0f:66:73:a8.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@casdev-srv01.newschool.edu's password: (enter remote server password)

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'casdev-srv01.newschool.edu'"
and check to make sure that only the key(s) you wanted were added.
casdev-master#  
```

to copy it to ***casdev-srv01***. Repeat the `ssh-copy-id` step to copy the key to each of the other servers in the environment (***casdev-srv02***, ***casdev-srv03***, ***casdev-casapp***, and ***casdev-samlsp***).

{% include reflinks.md %}
{% include links.html %}
