---
title: Configure time synchronization
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_configure-time-synchronization.html
summary:
---

For an active-active, multiple-server environment such as the one we're building to work properly, the time-of-day clocks on all servers in the environment must be in agreement. The Network Time Protocol (NTP) is used to ensure that each server is synchronized to Coordinated Universal Time (UTC).

{% include note.html content="The steps in this section should be performed on all six servers in the environment." %}

## Determine if NTP is already in use

RHEL 7 offers two NTP implementations, `ntpd` and `chronyd`. Run the commands

```console
# systemctl status chronyd
# systemctl status ntpd
```

to determine whether either of these is already in use. If output similar to this:

```
● chronyd.service - NTP client/server
    Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled)
    Active: active (running) since Ddd YYYY-MM-DD HH:MM:SS EDT; 58s ago
Main PID: 2530 (chronyd)
    CGroup: /system.slice/chronyd.service
            └─2530 /usr/sbin/chronyd -u chrony
```

or this:

```
● ntpd.service - Network Time Service
   Loaded: loaded (/usr/lib/systemd/system/ntpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Ddd YYYY-MM-DD HH:MM:SS EDT; 58s ago
  Process: 25086 ExecStart=/usr/sbin/ntpd -u ntp:ntp $OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 25087 (ntpd)
   CGroup: /system.slice/ntpd.service
           └─25087 /usr/sbin/ntpd -u ntp:ntp -g
```

appears, then one service or the other is installed and running, and nothing further needs to be done (go to the next section, [Install Apache Tomcat on the CAS servers][setup_tomcat_overview]). On the other hand, if output similar to this:

```
● ntpd.service - Network Time Service
   Loaded: loaded (/usr/lib/systemd/system/ntpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
```

or this:

```
Unit chronyd.service could not be found.
```

appears for both commands, then time is not being synchronized and NTP needs to be installed on each server in the development environment by following the remaining steps in this section.

## Install NTP (`ntpd`)

Generally, `ntpd` is preferred for always-on systems like servers, while `chronyd` is intended for use on systems like laptops that are shut down frequently or connected only intermittently to a network. Run the command

```console
# yum -y install ntp
```

to install `ntpd`.

## Configure `/etc/ntp.conf`

Edit the file `/etc/ntp.conf` and replace its entire contents with the `ntpd` configuration used by your organization. If your organization doesn't have a standard configuration, use something like the following example, which makes use of public time servers from the NTP Pool Project:

```
#
# Network Time Protocol configuration file (/etc/ntp.conf)
#
# Use this configuration file on Stratum 3 Linux systems.
#

#
# Stratum 2 servers. The total number of servers listed should be at least 2 more
# than the number specified for minclock and minsane, below.
#
server          0.us.pool.ntp.org               iburst
server          1.us.pool.ntp.org               iburst
server          2.us.pool.ntp.org               iburst
server          3.us.pool.ntp.org               iburst
server          0.north-america.pool.ntp.org    iburst
server          1.north-america.pool.ntp.org    iburst
server          2.north-america.pool.ntp.org    iburst
server          3.north-america.pool.ntp.org    iburst

#
# At least minsane candidate servers must be available for selection, and the
# mitigation algorithm must produce at least minclock candidates. Byzantine
# agreement principles require at least 4 candidates to correctly discard a
# single falseticker.
#
# http://support.ntp.org/bin/view/Support/StartingNTP4#Section_7.1.4.3.1.
#
tos minsane     4
tos minclock    4

#
# File used to record the frequency of the local clock oscillator. This is
# used at startup to set the initial frequency.
#
driftfile       /var/lib/ntp/drift
```

If you're not in the United States or North America, consult the NTP Pool Project's [pool server lists][ntp-pool-lists] for a list of servers in your country or on your continent.

## Open the NTP port in the firewall

In order to synchronize time with other systems, `ntpd` needs to be able to communicate on UDP port 123. RHEL 7's `firewalld` includes a pre-defined service for `ntp` to enable this. Run the commands

```console
# firewall-cmd --zone=public --add-service=ntp --permanent
success
# firewall-cmd --reload
success
#  
```

to open that service in the system firewall.

## Enable and start `ntpd`

Run the commands

```console
# systemctl enable ntpd
Created symlink from /etc/systemd/system/multi-user.target.wants/ntpd.service to /usr/lib/systemd/system/ntpd.service.
# systemctl start ntpd
```

to enable and start `ntpd`. After about 15-20 minutes, the protocol should have selected a server to synchronize with, and its status can be checked with the `ntpstat` and/or `ntpq` commands:

```console
# ntpstat
synchronised to NTP server (208.75.89.4) at stratum 3
   time correct to within 72 ms
   polling server every 1024 s
# ntpq -p localhost
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
+ns20.alltraders 127.67.113.92    2 u  713 1024  377   82.958   -2.849   1.783
-four10.gac.edu  18.26.4.105      2 u  403 1024  377   35.805   -0.444   2.610
-barry.tsi.io    198.60.22.240    2 u  352 1024  377   88.362   -0.088   2.111
-mdnworldwide.co 127.67.113.92    2 u  412 1024  371   52.765    2.509   4.455
-static-96-244-9 192.168.10.254   2 u  268 1024  377   10.932    1.255   3.564
+srcf-ntp.stanfo 171.64.7.105     2 u  219 1024  377   82.896   -3.092   1.710
*time.tritn.com  198.60.22.240    2 u  530 1024  377   68.547   -1.660   3.617
+bindcat.fhsu.ed 132.163.4.103    2 u  916 1024  377   58.940   -2.841   2.145
#  
```

{% include reflinks.md %}
{% include links.html %}
