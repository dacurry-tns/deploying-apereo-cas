---
title: Install the MongoDB software
last_updated: December 14, 2017
sidebar: main_sidebar
permalink: high-avail_mongodb_install-the-mongodb-software.html
summary:
---

Red Hat does not offer an up-to-date version of MongoDB on RHEL 7, but MongoDB, Inc. offers its own set of repositories from which the most current version can be installed.

{% include note.html content="Although the master build server will not be a member of the replica set, the MongoDB software will be installed there to facilitate customizing configuration files and distributing them to the replica set members, and to enable use of the `mongo` shell client application. In each section below, instructions are provided to indicate on which server(s) that step should be performed." %}

## Install the MongoDB repository

As of this writing the latest, stable version of MongoDB is 3.6, originally released in December 2017. To install MongoDB 3.6 with `yum`, create the file `/etc/yum.repos.d/mongodb-org-3.6.repo` on the master build server (***casdev-master***) with the following contents:

```
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```

Then copy the new file to each of the CAS servers by running the commands

```console
casdev-master# for i in 01 02 03
> do
> scp -p /etc/yum.repos.d/mongodb-org-3.6.repo casdev-srv${i}:/etc/yum.repos.d/mongodb-org-3.6.repo
> done
mongodb-org-3.6.repo                          100%  200   273.9KB/s   00:00
mongodb-org-3.6.repo                          100%  200   251.1KB/s   00:00
mongodb-org-3.6.repo                          100%  200   255.4KB/s   00:00
casdev-master#  
```

## Install MongoDB

Run the command

```console
# yum -y install mongodb-org
```

on the master build server (***casdev-master***) and each of the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) to install MongoDB.

## Correct directory permissions

The default MongoDB installation creates the MongoDB data directory and MongoDB log directory with permissions that allow all users on the system to access them. However, only the `mongod` user and `mongod` group actually need access to these directories, so access for other users should be removed. Run the command

```console
# chmod -R o= /var/lib/mongo /var/log/mongodb
```

on the master build server (***casdev-master***) and each of the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) to correct the directory permissions.

## Configure `logrotate`

By default, `mongod` will keep writing to the same log file until it is told not to; there is no pre-configured scheme for rotating logs. To correct this, configure the `logroate` program, which is probably already being used to rotate various system log files, to rotate `/var/log/mongodb/mongod.log` as well.

Create the file `/etc/logrotate.d/mongod` on the master build server (***casdev-master***) with the following contents:

```
/var/log/mongodb/mongod.log
{
    daily
    dateext
    dateformat -%Y-%m-%d
    dateyesterday
    extension .log
    missingok
    notifempty
    rotate 30
    sharedscripts
    postrotate
        /bin/kill -SIGUSR1 `cat /var/lib/mongo/mongod.lock 2> /dev/null` 2> /dev/null || true
    endscript
}
```

This will rotate `mongod.log` every day, moving the current log file to `mongod-YYYY-MM-DD.log`. Since `logrotate` typically runs in the wee hours of the morning, the `dateyesterday` directive tells it to use yesterday's date for the file name, since that's when most of the log entries will come from. The last 30 days' worth of log files will be kept; older files will be deleted automatically. Once the file has been rotated, `logrotate` will signal `mongod` to switch to the new log file.

Run the command

```console
casdev-master# logrotate -df /etc/logrotate.d/mongod
reading config file /etc/logrotate.d/mongod
extension is now .log
Allocating hash table for state file, size 15360 B

Handling 1 logs

rotating pattern: /var/log/mongodb/mongod.log
 forced from command line (30 rotations)
empty log files are not rotated, old logs are removed
considering log /var/log/mongodb/mongod.log
  log needs rotating
rotating log /var/log/mongodb/mongod.log, log->rotateCount is 30
Converted ' -%Y-%m-%d' -> '-%Y-%m-%d'
dateext suffix '-2017-12-13'
glob pattern '-[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]'
glob finding old rotated logs failed
fscreate context set to system_u:object_r:mongod_log_t:s0
renaming /var/log/mongodb/mongod.log to /var/log/mongodb/mongod-2017-12-13.log
running postrotate script
running script with arg /var/log/mongodb/mongod.log
: "
        /bin/kill -SIGUSR1 `cat /var/lib/mongo/mongod.lock 2> /dev/null` 2> /dev/null || true
"
casdev-master#  
```

to check the syntax of the file and confirm that it will do what you expect. Then copy the new file to each of the CAS servers by running the commands

```console
casdev-master# for i in 01 02 03
> do
> scp -p /etc/logrotate.d/mongod casdev-srv${i}:/etc/logrotate.d/mongod
> done
mongod                                        100%  293   400.8KB/s   00:00
mongod                                        100%  293   363.6KB/s   00:00
mongod                                        100%  293   397.7KB/s   00:00
casdev-master#  
```

## Disable the `mongod` service on the master build server

Since the master build server will not be part of the replica set, it does not need to run the MongoDB server (`mongod`). Run the command

```console
casdev-master# systemctl disable mongod
Removed symlink /etc/systemd/system/multi-user.target.wants/mongod.service.
casdev-master#  
```

on the master build server (***casdev-master***) to prevent `mongod` from being started when the system boots.

## References

[MongoDB: Install MongoDB Community Edition on Red Hat Enterprise or CentOS Linux][mongodb-install]

{% include reflinks.md %}
