---
title: Create a Maven WAR overlay project
last_updated: October 11, 2018
sidebar: main_sidebar
permalink: building_svcmgmt_create-a-maven-war-overlay-project.html
summary:
---

The CAS project provides a separate Maven WAR overlay template project for building the management webapp. We will use that as the starting point for our project.

## Clone the overlay template project

Use Git to clone the overlay template project from GitHub. Run the commands

```console
casdev-master# cd /opt/workspace
casdev-master# git clone https://github.com/apereo/cas-management-overlay.git
Cloning into 'cas-management-overlay'...
remote: Counting objects: 297, done.
remote: Compressing objects: 100% (15/15), done.
remote: Total 297 (delta 8), reused 14 (delta 4), pack-reused 276
Receiving objects: 100% (297/297), 132.54 KiB | 0 bytes/s, done.
Resolving deltas: 100% (152/152), done.
casdev-master#  
```

on the master build server (***casdev-master***). This will make a local copy of all files in the template project and store them in a directory called `cas-management-overlay`. It will also record the information needed for Git to keep the local copy of the files synchronized with the copy stored on GitHub, so that corrections and updates made by the project team can be incorporated into our project from time to time.

{% include tip.html content="As an alternative to using Git to clone a repository, GitHub allows the files in a repository to be downloaded in a Zip archive. However, this method does not include the metadata that Git needs to keep the local copy in sync with the master repository." %}

## Switch to the right branch

The GitHub repository for the overlay template project contains multiple versions of the template; each version is stored as a separate branch of the project. The `master` branch usually points to the version of the template used for configuring and deploying the latest stable release of the CAS server; this is the branch that will initially be copied to disk by cloning the project. Run the commands

```console
casdev-master# cd cas-management-overlay
casdev-master# grep '<cas.version>' pom.xml
        <cas.version>5.2.0</cas.version>
casdev-master#  
```

to determine which version of the CAS server the `master` branch will build. In most circumstances (including this project), the `master` branch of the template is the one you want to use (skip ahead to the next section, [Create a local branch][building_svcmgmt_create-a-maven-war-overlay-project.html#localbranch]).

If the `master` version of the template isn't for the version of the CAS server you want to work with (for example, if you want to work with an older version, or experiment with the version currently under development), run the command

```console
casdev-master# git branch -a
* master
  remotes/origin/4.1
  remotes/origin/4.2
  remotes/origin/5.0
  remotes/origin/5.1
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
casdev-master#  
```

to obtain a list of available branches, and then run the `git checkout` command to switch to that branch. For example, to switch back to the `5.1` branch, run the command

```console
casdev-master# git checkout 5.1
Branch 5.1 set up to track remote branch 5.1 from origin.
Switched to a new branch '5.1'
casdev-master#  grep '<cas.version>' pom.xml
        <cas.version>5.1.5</cas.version>
casdev-master#  
```

to switch branches (it's not necessary to type the `remotes/origin/` part of the branch name). This will download additional/changed files from GitHub to the local disk. You can switch back to the current version of the template by checking out the `master` branch again:

```console
casdev-master# git checkout master
Switched to branch 'master'
casdev-master#  grep '<cas.version>' pom.xml
        <cas.version>5.2.0</cas.version>
casdev-master#  
```

## Create a local branch {#localbranch}

After you're on the right branch (for our project, you should be on the `master` branch), create a new branch local to your project, which will be used to track all of your changes and keep them separate from any changes made to the template by the CAS developers. This will make it easier in the future to merge upstream changes from the CAS project team into your local template without having to redo all your changes.

Choose a meaningful name for your branch, but not something likely to be duplicated by the CAS developers&mdash;for example, `newschool-casdev`. Run the commands

```console
casdev-master# git checkout -b newschool-casdev
Switched to a new branch 'newschool-casdev'
casdev-master#  
```

to create this new branch (replace `newschool-casdev` with the name of your branch).

{% include links.html %}
