---
title: Set up MongoDB authentication
last_updated: December 20, 2017
sidebar: main_sidebar
permalink: high-avail_mongodb_set-up-mongodb-authentication.html
summary:
---

MongoDB provides an internal authentication feature that, when enabled, will require the individual members of the replica set to authenticate to each other. MongoDB also provides role-based access control, which requires client applications (and users) to authenticate to the database with a username and password, and then sets limits on the database(s) each user may access, and the operations the user may perform there. Both of these features will be used to protect the data stored in the CAS MongoDB instance.

## Create an administrative user

On one of the replica set members (e.g., ***casdev-srv01***), start `mongod` by running the commands

```console
casdev-srv01# systemctl start mongod-disable-thp
casdev-srv01# systemctl start mongod
```

{% include note.html content="Make sure you are using one of the replica set members (CAS servers), not the master build server (***casdev-master***)." %}

On the same server, run the `mongo` shell to connect to the server:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">casdev-srv01# </span><span class="nc">mongo</span><span class="kv">
</span>MongoDB shell version v3.6.0
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.6.0
Server has startup warnings:
YYYY-MM-DDTHH:MM:SS.sss-0000 I CONTROL  [initandlisten]
YYYY-MM-DDTHH:MM:SS.sss-0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
YYYY-MM-DDTHH:MM:SS.sss-0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
YYYY-MM-DDTHH:MM:SS.sss-0000 I CONTROL  [initandlisten]
<span class="ni">&gt; </span>
</code></pre>
</div>

Administrative users are created in the special `admin` database. Using the `mongo` shell, connect to the `admin` database and then create an administrative user called `mongoadmin` by running the commands

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">&gt; </span><span class="nc">use</span><span class="kv"> admin</span>
switched to db admin
<span class="ni">&gt; </span><span class="nc">db.createUser( { user: "mongoadmin", pwd: "changeit", roles: [ { role: "root", db: "admin" } ] } )</span>
Successfully added user: {
        "user" : "mongoadmin",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ]
}
<span class="ni">&gt;  </span>
</code></pre>
</div>

{% include warning.html content="The command above uses `changeit` as the value of the `mongoadmin` password. Obviously, something other than this should be used in a production MongoDB deployment." %}

Then exit the `mongo` shell:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">&gt; </span><span class="nc">exit</span>
bye
</code></pre>
</div>

## Generate a SCRAM-SHA1 keyfile

To implement internal authentication between the replica set members, MongoDB supports the [Salted Challenge Response Authentication Mechanism (SCRAM-SHA-1)][scram]. To support this, a keyfile containing the shared secret (password) is created and installed on each replica set member server. Run the command

```console
casdev-master# openssl rand -base64 756 > mongod-auth.key
```

on the master build server (***casdev-master***) to generate a random key (password). Although the master build server is not a member of the replica set, it makes sense to store a copy of the keyfile there for safekeeping. Then run the commands

```console
casdev-master# tar cf kf.tar --owner=mongod --group=mongod --mode=400 mongod-auth.key
casdev-master# for i in 01 02 03
> do
> scp kf.tar casdev-srv${i}:/tmp/kf.tar
> ssh casdev-srv${i} "cd /var/lib/mongo; tar xf /tmp/kf.tar; rm /tmp/kf.tar"
> done
kf.tar                                        100%   10KB 437.3KB/s   00:00
kf.tar                                        100%   10KB   1.0MB/s   00:00
kf.tar                                        100%   10KB 128.4KB/s   00:00
casdev-master#  
```

to distribute the keyfile to each of the replica set members with the correct owner, group, and permissions.

## Update the MongoDB configuration file

MongoDB uses a YAML-formatted configuration file, `/etc/mongod.conf`. Edit this file on the master build server (***casdev-master***) and make the following changes:

1. In the `net` section, change the value of the `bindIp` setting from `127.0.0.1` (listen only on the loopback interface) to `0.0.0.0` (listen on all interfaces). This will enable the other members of the replica set to connect to the server.
2. Uncomment the `security` section and add a `keyFile` setting with the path to the keyfile created above (`/var/lib/mongo/mongod-auth.key`).
3. Also in the `security` section, add an `authorization` setting with the value `enabled` (this turns on role-based access control).
4. Uncomment the `replication` section and add a `replSetName` setting with the value `rs0`.

After making these changes, the affected sections of the configuration file should look like this:

```yaml
net:
  port: 27017
  bindIp: 0.0.0.0

security:
  keyFile: /var/lib/mongo/mongod-auth.key
  authorization: enabled

replication:
  replSetName: rs0
```

Then run the commands

```console
casdev-master# for i in 01 02 03
> do
> scp -p /etc/mongod.conf casdev-srv${i}:/etc/mongod.conf
> ssh casdev-srv${i} "systemctl start mongod-disable-thp; systemctl restart mongod"
> done
mongod.conf                                   100%  813    41.2KB/s   00:00
mongod.conf                                   100%  813    53.4KB/s   00:00
mongod.conf                                   100%  813   721.3KB/s   00:00
casdev-master#  
```

to copy the updated configuration file to each member of the replica set and (re)start the `mongod` server.

## References

* [MongoDB: Internal Authentication][mongodb-internal-auth]
* [MongoDB: Role-Based Access Control][mongodb-rbac]

{% include reflinks.md %}
