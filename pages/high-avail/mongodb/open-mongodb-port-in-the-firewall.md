---
title: Open MongoDB port in the firewall
last_updated: December 14, 2017
sidebar: main_sidebar
permalink: high-avail_mongodb_open-mongodb-port-in-the-firewall.html
summary:
---

To enable the `mongod` processes in the replica set to communicate with each other, the MongoDB port (TCP 27017) must be opened in the firewall on each of the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***).

## Create a `firewalld` service configuration

First, create a `firewalld` service configuration file on the master build server (***casdev-master***) called `/etc/firewalld/services/mongod.xml` with the following contents:

```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>mongod</short>
  <description>MongoDB default port for mongod and mongos instances.</description>
  <port protocol="tcp" port="27017"/>
</service>
```

to define the service, and then run the commands

```console
casdev-master# restorecon /etc/firewalld/services/mongod.xml
casdev-master# chmod 640 /etc/firewalld/services/mongod.xml
casdev-master# firewall-cmd --reload
success
casdev-master#  
```

to assign the correct SELinux context and file permissions to the `mongod.xml` file and inform `firewalld` of its existence. Then copy the new file to each of the CAS servers and inform their `firewalld` processes of its existence by running the commands

```console
casdev-master# for i in 01 02 03
> do
> scp -p /etc/firewalld/services/mongod.xml casdev-srv${i}:/etc/firewalld/services/mongod.xml
> ssh casdev-srv${i} firewall-cmd --reload
> done
mongod.xml                                    100%  205   309.3KB/s   00:00
success
mongod.xml                                    100%  205   320.6KB/s   00:00
success
mongod.xml                                    100%  205   333.8KB/s   00:00
success
casdev-master#  
```

## Configure the firewall

Because some of the information stored in MongoDB may be sensitive (e.g., passwords in configuration properties), we will only open the MongoDB port in the firewall to connections from the CAS servers and the master build server.

### Create an ipset of source addresses

A `firewalld` *ipset* is a named list of IP addresses that can be referenced in firewall rules. We will define an ipset called `cas-servers` that contains the addresses of the master build server and the CAS servers, which can be used to create the firewall rule in the next section. Run the commands

```console
# firewall-cmd  --permanent --new-ipset=cas-servers --type=hash:net
success
# firewall-cmd --reload
success
# firewall-cmd --permanent --ipset=cas-servers --add-entry=192.168.100.100
success
# firewall-cmd --permanent --ipset=cas-servers --add-entry=192.168.100.101
success
# firewall-cmd --permanent --ipset=cas-servers --add-entry=192.168.100.102
success
# firewall-cmd --permanent --ipset=cas-servers --add-entry=192.168.100.103
success
# firewall-cmd --reload
success
#  
```

on each of the three CAS servers (***casdev-srv01***, ***casdev-srv02***. and ***casdev-srv03***). It is not necessary to define the ipset on the master build server (***casdev-master***), since it will not be running `mongod`.

### Create a rich rule to enable access

In addition to command-line arguments that allow the creation of basic allow/deny rules, `firewalld` supports a *rich rule* language for creating more complex rules. The rich language extends the basic set of elements (service, port, etc.) with additional elements, such as source and destination addresses, logging, actions and limits for logs and actions. We will use a rich rule to limit connections to the `mongod` port to the IP addresses in the `cas-servers` ipset defined above. Run the commands

```console
# firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source ipset="cas-servers" service name="mongod" accept'
success
# firewall-cmd --reload
success
#  
```
on each of the three CAS servers (***casdev-srv01***, ***casdev-srv02***. and ***casdev-srv03***). It is not necessary to install the rule on the master build server (***casdev-master***), since it will not be running `mongod`.

## References

* [Firewalld: IP Sets][firewalld-ipset]
* [Firewalld: Rich Rule Language][firewalld-richlanguage]

{% include reflinks.md %}
