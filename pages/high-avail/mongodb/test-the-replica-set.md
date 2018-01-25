---
title: Test the replica set
last_updated: December 20, 2017
sidebar: main_sidebar
permalink: high-avail_mongodb_test-the-replica-set.html
summary:
---

To verify that replication is working, we will create some test data on the primary member, and then try to read it from one (or more) of the secondary members.

## Connect to the primary with the `mongo` shell

On the primary replicat set member (as determined by the output from `rs.staus()` in the previous section), start the `mongo` shell again by running the command

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">casdev-srv01# </span><span class="nc">mongo</span><span class="kv"> -u mongoadmin -p --authenticationDatabase admin
</span>MongoDB shell version v3.6.0
<span class="ni">Enter password:</span>
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.6.0
<span class="ni">rs0:PRIMARY&gt; </span>
</code></pre>
</div>

and entering the correct password ("`changeit`"). The `mongo` shell prompt now displays the replica set name and member status.

## Create some test data

Enter the commands below to create a new database called `testDatabase` (databases are created automatically the first time they are used) and store some documents in a collection called `testCollection`:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">rs0:PRIMARY&gt; </span><span class="nc">use</span><span class="kv"> testDatabase</span>
switched to db testDatabase
<span class="ni">rs0:PRIMARY&gt; </span><span class="nc">for</span><span class="kv"> (var i=1; i &lt;= 10; i++) db.testCollection.insert( { val: i } )</span>
WriteResult({ "nInserted" : 1 })
<span class="ni">rs0:PRIMARY&gt; </span>
</code></pre>
</div>

Then retrieve the documents just created by running the command

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">rs0:PRIMARY&gt; </span><span class="nc">db.testCollection.find()</span>
{ "_id" : ObjectId("5a298797050075eeef5df805"), "val" : 1 }
{ "_id" : ObjectId("5a298797050075eeef5df806"), "val" : 2 }
{ "_id" : ObjectId("5a298797050075eeef5df807"), "val" : 3 }
{ "_id" : ObjectId("5a298797050075eeef5df808"), "val" : 4 }
{ "_id" : ObjectId("5a298797050075eeef5df809"), "val" : 5 }
{ "_id" : ObjectId("5a298797050075eeef5df80a"), "val" : 6 }
{ "_id" : ObjectId("5a298797050075eeef5df80b"), "val" : 7 }
{ "_id" : ObjectId("5a298797050075eeef5df80c"), "val" : 8 }
{ "_id" : ObjectId("5a298797050075eeef5df80d"), "val" : 9 }
{ "_id" : ObjectId("5a298797050075eeef5df80e"), "val" : 10 }
<span class="ni">rs0:PRIMARY&gt; </span>
</code></pre>
</div>

## Connect to a secondary with the `mongo` shell

Now connect with the `mongo` shell on one of the secondary members (e.g., ***casdev-srv02***) by running the command

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">casdev-srv02# </span><span class="nc">mongo</span><span class="kv"> -u mongoadmin -p --authenticationDatabase admin
</span>MongoDB shell version v3.6.0
<span class="ni">Enter password:</span>
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.6.0
<span class="ni">rs0:SECONDARY&gt;</span>
</code></pre>
</div>

and entering the correct password ("`changeit`").

## Enable secondary member read operations

By default, reads from secondary members are not allowed; this is to ensure that a query does not retrieve stale data. Even simple commands like `show dbs` and `show collections` will fail with an error. But in this case we want to read from the secondary, to make sure the data got copied from the primary. To enable this, run the command

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">rs0:SECONDARY&gt; </span><span class="nc">db.getMongo().setSlaveOk()</span>
</code></pre>
</div>

which will enable reading from the secondary for duration of this connection.

## Check that everything was replicated

Run the commands

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">rs0:SECONDARY&gt; </span><span class="nc">show</span><span class="kv"> dbs</span>
admin         0.000GB
config        0.000GB
local         0.000GB
testDatabase  0.000GB
<span class="ni">rs0:SECONDARY&gt; </span><span class="nc">use</span><span class="kv"> testDatabase</span>
switched to db testDatabase
<span class="ni">rs0:SECONDARY&gt; </span><span class="nc">show</span><span class="kv"> collections</span>
testCollection
<span class="ni">rs0:SECONDARY&gt; </span>
</code></pre>
</div>

to verify that `testDatabase` and `testCollection` are indeed available on the secondary. Then run the command

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">rs0:SECONDARY&gt; </span><span class="nc">db.testCollection.find()</span>
{ "_id" : ObjectId("5a298797050075eeef5df805"), "val" : 1 }
{ "_id" : ObjectId("5a298797050075eeef5df807"), "val" : 3 }
{ "_id" : ObjectId("5a298797050075eeef5df808"), "val" : 4 }
{ "_id" : ObjectId("5a298797050075eeef5df806"), "val" : 2 }
{ "_id" : ObjectId("5a298797050075eeef5df80a"), "val" : 6 }
{ "_id" : ObjectId("5a298797050075eeef5df80c"), "val" : 8 }
{ "_id" : ObjectId("5a298797050075eeef5df80d"), "val" : 9 }
{ "_id" : ObjectId("5a298797050075eeef5df80b"), "val" : 7 }
{ "_id" : ObjectId("5a298797050075eeef5df80e"), "val" : 10 }
{ "_id" : ObjectId("5a298797050075eeef5df809"), "val" : 5 }
<span class="ni">rs0:SECONDARY&gt; </span>
</code></pre>
</div>

to check that, indeed, the data inserted on the primary is also present on the secondary. The data may not appear in numerical order, since the command did not attempt to sort them, but they should all be present.

## Delete the test database

Exit the `mongo` shell on the secondaries, and then run the commands

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">rs0:PRIMARY&gt; </span><span class="nc">show </span><span class="kv">dbs</span>
admin         0.000GB
config        0.000GB
local         0.000GB
testDatabase  0.000GB
<span class="ni">rs0:SECONDARY&gt; </span><span class="nc">use</span><span class="kv"> testDatabase</span>
switched to db testDatabase
<span class="ni">rs0:SECONDARY&gt; </span><span class="nc">db.dropDatabase()</span>
{
        "dropped" : "testDatabase",
        "ok" : 1,
        "operationTime" : Timestamp(1512671689, 2),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1512671689, 2),
                "signature" : {
                        "hash" : BinData(0,"CpHBrLKG56+ehaMf8Uk5QlzeSX8="),
                        "keyId" : NumberLong("6496845223040122881")
                }
        }
}
<span class="ni">rs0:PRIMARY&gt; </span>
</code></pre>
</div>

on the primary to remove the test database and its contents.

## References

* [Linode: Create a MongoDB Replica Set][linode-mongodb-replica]

{% include reflinks.md %}
