---
title: Disable Transparent Huge Pages
last_updated: December 14, 2017
sidebar: main_sidebar
permalink: high-avail_mongodb_disable-transparent-huge-pages.html
summary:
---

{% include note.html content="This step is only necessary when running MongoDB on Linux servers (physical or virtual)." %}

Transparent Huge Pages (THP) is a Linux memory management feature designed to reduce the overhead of translation lookaside buffer (page table) lookups on machines with large amounts of memory by using larger virtual memory pages. However, THP often causes performance problems for database workloads, because they tend to have sparse, rather than contiguous, memory access patterns.  MongoDB, Inc. recommends disabling THP on servers running MongoDB.

## Define a service unit to disable THP

Create a file on the master build server (***casdev-master***) called `/etc/systemd/system/mongod-disable-thp.service` with the following contents:

```
[Unit]
Description="Disable Transparent Huge Pages (THP) before mongod starts"
Before=mongod.service

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/enabled'
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/defrag'

[Install]
RequiredBy=mongod.service
```

## Install and enable the service unit

Run the commands

```console
casdev-master# restorecon /etc/systemd/system/mongod-disable-thp.service
casdev-master# chmod 644 /etc/systemd/system/mongod-disable-thp.service
casdev-master# for i in 01 02 03
> do
> scp -p /etc/systemd/system/mongod-disable-thp.service casdev-srv${i}:/etc/systemd/system/mongod-disable-thp.service
> ssh casdev-srv${i} systemctl enable mongod-disable-thp
> done
mongod-disable-thp.service                    100%  321   984.7KB/s   00:00
Created symlink from /etc/systemd/system/mongod.service.requires/mongod-disable-thp.service to /etc/systemd/system/mongod-disable-thp.service.
mongod-disable-thp.service                    100%  321     1.0MB/s   00:00
Created symlink from /etc/systemd/system/mongod.service.requires/mongod-disable-thp.service to /etc/systemd/system/mongod-disable-thp.service.
mongod-disable-thp.service                    100%  321   441.2KB/s   00:00
Created symlink from /etc/systemd/system/mongod.service.requires/mongod-disable-thp.service to /etc/systemd/system/mongod-disable-thp.service.
casdev-master#  
```

to copy the service unit to the CAS servers and enable it to be executed by `systemd`. Do not enable the service unit on the master build server (***casdev-master***), since `mongod` will not be running there.

## References

[MongoDB: Disable Transparent Huge Pages (THP)][mongodb-disable-thp]

{% include reflinks.md %}
{% include links.html %}
