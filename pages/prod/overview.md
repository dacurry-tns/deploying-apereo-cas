---
title: Moving to production
last_updated: October 18, 2018
sidebar: main_sidebar
permalink: prod_overview.html
summary: Moving from development and testing to production requires some additional steps to make sure everything goes smoothly.
---

The New School CAS 5 environment entered production over the University's 2018 Spring Break week.

The environment is essentially the one [described in the introduction][introduction_sso-environment-architecture], with a total of five virtual servers (two in one data center, three in the other) operating in a pool behind a pair of F5 load balancers (one in each data center, in an active/passive configuration). Each virtual server is running a Tomcat instance (running both the CAS server and the CAS management webapp) and a MongoDB instance. The MongoDB instances are all members of the same replica set (which is why there are five servers; replica sets require an odd number of members) and handle the distributed ticket registry and distributed service registry.

The servers manage access to approximately 50 applications, hosted both locally in our data centers and remotely in the cloud by various Software-as-a-Service providers. Half a dozen of these applications are SAML2-based and authenticate through the CAS SAML2 IdP; the rest are CAS-based. Most of the applications require only the user principal name (username) or a single user attribute, although a few require more.

**Event counts for Oct. 1 - Oct. 15, 2018**

| Event | Average Events/Day |
| ----- | -----: |
| Authentication Event Triggered | 67,183 |
| Authentication Success | 21,905 |
| Authentication Failed | 2,126  |
| Service Ticket Created | 32,612 |
| Service Ticket Not Created | 15 |
| Service Ticket Validated | 22,224 |
| Service Ticket Validate Failed | 509  |
| Ticket Granting Ticket Created | 21,457 |
| Ticket Granting Ticket Destroyed | 10,109 |

This chapter describes steps and configuration changes that should be considered when moving from a development and/or test environment to a production environment. It also describes some of the problems we have encountered after going live, and how we corrected them.

{% include reflinks.md %}
{% include links.html %}
