---
title: Create the replica set
last_updated: January 25, 2018
sidebar: main_sidebar
permalink: high-avail_mongodb_create-the-replica-set.html
summary:
---

The replica set is created by initiating replication on the replica set member server where the administrative user was created, and then adding the other replica set member servers to the set.

## Connect with the `mongo` shell

On the replica set member where the `mongoadmin` user was created in the previous section (***casdev-srv01*** in our case), start the `mongo` shell again by running the command

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">casdev-srv01# </span><span class="kv"> mongo -u mongoadmin -p --authenticationDatabase admin
</span>MongoDB shell version v3.6.0
<span class="ni">Enter password:</span>
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.6.0
<span class="ni">&gt; </span>
</code></pre>
</div>

and entering the correct password ("`changeit`").

## Initiate the replica set

From the `mongo` shell, run the command

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">&gt; </span><span class="nc">rs.initiate()</span>
{
        "info2" : "no configuration specified. Using a default configuration for the set",
        "me" : "casdev-srv01.newschool.edu:27017",
        "ok" : 1,
        "operationTime" : Timestamp(1512664653, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1512664653, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
<span class="ni">&gt; </span>
</code></pre>
</div>

## Add members to the replica set

Continuing in the `mongo` shell, add the other members of the replica set:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">&gt; </span><span class="nc">rs.add(</span><span class="kv">"casdev-srv02.newschool.edu"</span><span class="nc">)</span>
{
        "ok" : 1,
        "operationTime" : Timestamp(1512664727, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1512664727, 1),
                "signature" : {
                        "hash" : BinData(0,"wbhbHdhI1gtR+SWQSh2XARQw9jw="),
                        "keyId" : NumberLong("6496845223040122881")
                }
        }
}
<span class="ni">&gt; </span><span class="nc">rs.add(</span><span class="kv">"casdev-srv03.newschool.edu"</span><span class="nc">)</span>
{
        "ok" : 1,
        "operationTime" : Timestamp(1512664876, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1512664876, 1),
                "signature" : {
                        "hash" : BinData(0,"gIrttrr3VzqhM2iIWxX5ITwMIhI="),
                        "keyId" : NumberLong("6496845223040122881")
                }
        }
}
<span class="ni">&gt;  </span>
</code></pre>
</div>

## Display the replica set configuration

To view the configuration of the replica set, use the `rs.conf()` command to the `mongo` shell:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">&gt; </span><span class="nc">rs.conf()</span>
{
        "_id" : "rs0",
        "version" : 3,
        "protocolVersion" : NumberLong(1),
        "members" : [
                {
                        "_id" : 0,
                        "host" : "casdev-srv01.newschool.edu:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 1,
                        "host" : "casdev-srv02.newschool.edu:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 2,
                        "host" : "casdev-srv03.newschool.edu:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5a296e4da9fdf50c1fc967ae")
        }
}
<span class="ni">&gt;  </span>
</code></pre>
</div>

## Display the status of the replica set

To display dynamic information about the status of the replica set, use the `rs.status()` command instead:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">&gt; </span><span class="nc">rs.status()</span>
{
        "set" : "rs0",
        "date" : ISODate("YYYY-MM-DDTHH:MM:SS.sssZ"),
        "myState" : 1,
        "term" : NumberLong(1),
        "heartbeatIntervalMillis" : NumberLong(2000),
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1512664965, 1),
                        "t" : NumberLong(1)
                },
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1512664965, 1),
                        "t" : NumberLong(1)
                },
                "appliedOpTime" : {
                        "ts" : Timestamp(1512664965, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1512664965, 1),
                        "t" : NumberLong(1)
                }
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "casdev-srv01.newschool.edu:27017",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 916,
                        "optime" : {
                                "ts" : Timestamp(1512664965, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("YYYY-MM-DDTHH:MM:SSZ"),
                        "electionTime" : Timestamp(1512664653, 2),
                        "electionDate" : ISODate("2017-12-07T16:37:33Z"),
                        "configVersion" : 3,
                        "self" : true
                },
                {
                        "_id" : 1,
                        "name" : "casdev-srv02.newschool.edu:27017",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 247,
                        "optime" : {
                                "ts" : Timestamp(1512664965, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1512664965, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("YYYY-MM-DDTHH:MM:SSZ"),
                        "optimeDurableDate" : ISODate("YYYY-MM-DDTHH:MM:SSZ"),
                        "lastHeartbeat" : ISODate("YYYY-MM-DDTHH:MM:SS.sssZ"),
                        "lastHeartbeatRecv" : ISODate("YYYY-MM-DDTHH:MM:SS.sssZ"),
                        "pingMs" : NumberLong(0),
                        "syncingTo" : "casdev-srv01.newschool.edu:27017",
                        "configVersion" : 3
                },
                {
                        "_id" : 2,
                        "name" : "casdev-srv03.newschool.edu:27017",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 99,
                        "optime" : {
                                "ts" : Timestamp(1512664965, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1512664965, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("YYYY-MM-DDTHH:MM:SSZ"),
                        "optimeDurableDate" : ISODate("YYYY-MM-DDTHH:MM:SSZ"),
                        "lastHeartbeat" : ISODate("YYYY-MM-DDTHH:MM:SS.sssZ"),
                        "lastHeartbeatRecv" : ISODate("YYYY-MM-DDTHH:MM:SS.sssZ"),
                        "pingMs" : NumberLong(0),
                        "syncingTo" : "casdev-srv01.newschool.edu:27017",
                        "configVersion" : 3
                }
        ],
        "ok" : 1,
        "operationTime" : Timestamp(1512664965, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1512664965, 1),
                "signature" : {
                        "hash" : BinData(0,"wGZmpqOqx1Xz1XDrsa2129JAd+c="),
                        "keyId" : NumberLong("6496845223040122881")
                }
        }
}
<span class="ni">&gt;  </span>
</code></pre>
</div>

The `members[n].stateStr` element indicates, for each member, whether it is the primary or a secondary member of the replica set.

## Display current replication status

To display the current status of replicating data from the primary to the slave (secondary) servers, use the `rs.printSlaveReplicationInfo()` command:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">&gt; </span><span class="nc">rs.printSlaveReplicationInfo()</span>
source: casdev-srv02.newschool.edu:27017
        syncedTo: Ddd MMM DD YYYY HH:MM:DD GMT-0500 (EST)
        0 secs (0 hrs) behind the primary
source: casdev-srv03.newschool.edu:27017
        syncedTo: Ddd MMM DD YYYY HH:MM:DD GMT-0500 (EST)
        0 secs (0 hrs) behind the primary
<span class="ni">&gt;  </span>
</code></pre>
</div>

## Exit the `mongo` shell

Exit the `mongo` shell:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">&gt; </span><span class="nc">exit</span>
bye
</code></pre>
</div>

## References

* [MongoDB: Deploy Replica Set With Keyfile Access Control][mongodb-replica-keyfile]
* [Linode: Create a MongoDB Replica Set][linode-mongodb-replica]

{% include reflinks.md %}
{% include links.html %}
