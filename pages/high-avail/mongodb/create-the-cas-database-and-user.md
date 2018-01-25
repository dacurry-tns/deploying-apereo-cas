---
title: Create the CAS database and user
last_updated: January 25, 2018
sidebar: main_sidebar
permalink: high-avail_mongodb_create-the-cas-database-and-user.html
summary:
---

We will create a CAS-specific database where the CAS server can store all its data. Each of the different modules (ticket registry, service registry, etc.) will store its information in a separate collection within this database. We will also create a "regular" user (one that does not have administrative rights) to be used by the CAS servers to access these tables.

## Create the CAS database

MongoDB does not have a special command to create a database. Rather, the database is created the first time it is used. To create the database, connect to the primary replica set member with the `mongo` shell and issue a `use <databasename>` command:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">casdev-master# </span><span class="nc">mongo</span><span class="kv"> -u mongoadmin -p --authenticationDatabase admin --ssl --host rs0/casdev-srv01.newschool.edu,casdev-srv02.newschool.edu,casdev-srv03.newschool.edu
</span>MongoDB shell version v3.6.0
<span class="ni">Enter password:</span>
connecting to: mongodb://casdev-srv01.newschool.edu:27017,casdev-srv02.newschool.edu:27017,casdev-srv03.newschool.edu:27017/?replicaSet=rs0
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Starting new replica set monitor for rs0/casdev-srv01.newschool.edu:27017,casdev-srv02.newschool.edu:27017,casdev-srv03.newschool.edu:27017
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Successfully connected to casdev-srv02.newschool.edu:27017 (1 connections now open to casdev-srv02.newschool.edu:27017 with a 5 second timeout)
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [ReplicaSetMonitor-TaskExecutor-0] Successfully connected to casdev-srv01.newschool.edu:27017 (1 connections now open to casdev-srv01.newschool.edu:27017 with a 5 second timeout)
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Successfully connected to casdev-srv03.newschool.edu:27017 (1 connections now open to casdev-srv03.newschool.edu:27017 with a 5 second timeout)
MongoDB server version: 3.6.0
<span class="ni">rs0:PRIMARY&gt; </span><span class="nc">use</span><span class="kv"> casdb</span>
switched to db casdb
<span class="ni">rs0:PRIMARY&gt; </span>
</code></pre>
</div>

This will create a database called `casdb`.

## Create a database user

Database users can be created in the `admin` database, or in the database they will be accessing. If the user is in a different database than the one being connected to however, then the connection command must specify the database to authenticate against. To simplify things, the CAS database user will be created in the `casdb` database created above. In the `mongo` shell, switch to the `casdb` database and create a new user by running the commands

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">rs0:PRIMARY&gt; </span><span class="nc">use</span><span class="kv"> casdb</span>
switched to db casdb
<span class="ni">rs0:PRIMARY&gt; </span><span class="nc">db.createUser(</span><span class="kv"> { user: "mongocas", pwd: "changeit", roles: [ { role: "readWrite", db: "casdb" } ] } </span><span class="nc">)</span>
Successfully added user: {
        "user" : "mongocas",
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "casdb"
                }
        ]
}
<span class="ni">rs0:PRIMARY&gt; </span>
</code></pre>
</div>

This will create a user named `mongocas` with password `changeit`. The user will have read/write access to the `casdb` database, and no access to any other database (this can be changed later, by adjusting the user's roles).

{% include warning.html content="The command above uses `changeit` as the value of the `mongocas` password. Obviously, something other than this should be used in a production MongoDB deployment." %}

## Test the new database and user

Test the new database and user by exiting the administrative `mongo` shell and running the command

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">rs0:PRIMARY&gt; </span><span class="nc">exit</span>
bye
<span class="ni">casdev-master# </span><span class="nc">mongo</span><span class="kv"> casdb -u mongocas -p --ssl --host rs0/casdev-srv01.newschool.edu,casdev-srv02.newschool.edu,casdev-srv03.newschool.edu
</span>MongoDB shell version v3.6.0
<span class="ni">Enter password:</span>
connecting to: mongodb://casdev-srv01.newschool.edu:27017,casdev-srv02.newschool.edu:27017,casdev-srv03.newschool.edu:27017/casdb?replicaSet=rs0
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Starting new replica set monitor for rs0/casdev-srv01.newschool.edu:27017,casdev-srv02.newschool.edu:27017,casdev-srv03.newschool.edu:27017
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Successfully connected to casdev-srv02.newschool.edu:27017 (1 connections now open to casdev-srv02.newschool.edu:27017 with a 5 second timeout)
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [ReplicaSetMonitor-TaskExecutor-0] Successfully connected to casdev-srv01.newschool.edu:27017 (1 connections now open to casdev-srv01.newschool.edu:27017 with a 5 second timeout)
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Successfully connected to casdev-srv03.newschool.edu:27017 (1 connections now open to casdev-srv03.newschool.edu:27017 with a 5 second timeout)
MongoDB server version: 3.6.0
<span class="ni">rs0:PRIMARY&gt; </span><span class="nc">exit</span>
bye
<span class="ni">casdev-master# </span><span class="kv">
</span></code></pre>
</div>

to connect to the `casdb` database as the `mongocas` user.

## References

* [MongoDB: Add Users][mongodb-add-users]

{% include reflinks.md %}
{% include links.html %}
