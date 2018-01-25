---
title: Install and configure MongoDB
last_updated: December 14, 2017
sidebar: main_sidebar
permalink: high-avail_mongodb_overview.html
summary: MongoDB will be used to store the ticket registry, service registry, and configuration properties for all CAS servers in the environment.
---

[MongoDB][mongodb-org] is an open-source NoSQL database. MongoDB stores data records as JSON-like *documents* that contain field-value pairs. The value of a field can be any of several different data types such as numbers, strings, booleans, dates, objects, and arrays. In MongoDB, *databases* hold *collections* of documents. Collections are somewhat analogous to tables in relational databases, but a collection does not require its documents to have the same schema; i.e. the documents in a single collection do not all have to have the same set of fields and the data type for a field can differ from one document to another within a collection.

MongoDB is a distributed database by design, so high availability, horizontal scaling, and geographic distribution are built in and easy to use. A *replica set* is a group of MongoDB instances that manage the same data set (group of databases). Replica sets provide redundancy and high availability, and are the basis for all production MongoDB deployments. A replica set contains multiple data storage nodes and, optionally, an arbiter node (used when needed to ensure there are an odd number of members in the replica set).

One and only one of the data storage nodes is deemed the primary node, and the others are deemed secondary nodes (or arbiters). The primary node receives all write operations (and usually, all read operations as well). The primary records all changes to its data set in a transaction log. The secondary nodes copy and apply these changes in an asynchronous process, resulting in the same data set being stored on multiple servers. If the primary server is unavailable, the secondaries will hold an election to elect one of themselves as the new primary.

In this section, we will install the latest, stable version of MongoDB on each of the CAS servers in the environment (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***), and then group those servers together as a replica set. We will also implement security controls to prevent access to the servers (and their data) from unauthorized sources, since some of the data stored in the database may be sensitive (e.g., passwords in configuration properties).

{% include reflinks.md %}
