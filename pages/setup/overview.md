---
title: Setting up the environment
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_overview.html
summary: Before beginning the CAS build and configuration process, the server environment should be prepared by creating virtual machines, installing necessary software dependencies, and performing basic software configuration and system administration tasks.
---

The New School's CAS 5 development environment is comprised of six servers, all in the ***newschool.edu*** domain. There is one master build server:

<table>
    <colgroup>
        <col width="25%" />
        <col width="75%" />
    </colgroup>
    <tbody>
        <tr>
            <td markdown="span">***casdev-master***<br />192.168.100.100</td>
            <td markdown="span">The master build server where software will be built for deployment to the other servers. This server will include development tools (compilers, libraries, etc.) that are not appropriate for installation on user-facing servers.</td>
        </tr>
    </tbody>
</table>

There is also a pool of three identical CAS servers:

<table>
    <colgroup>
        <col width="25%" />
        <col width="75%" />
    </colgroup>
    <tbody>
        <tr>
            <td markdown="span">***casdev-srv01***<br />192.168.100.101</td>
            <td markdown="span">A CAS server instance; a member of the F5 load balancers' server pool for the ***casdev.newschool.edu*** virtual address.</td>
        </tr>
        <tr>
            <td markdown="span">***casdev-srv02***<br />192.168.100.102</td>
            <td markdown="span">A CAS server instance; a member of the F5 load balancers' server pool for the ***casdev.newschool.edu*** virtual address.</td>
        </tr>
        <tr>
            <td markdown="span">***casdev-srv03***<br />192.168.100.103</td>
            <td markdown="span">A CAS server instance; a member of the F5 load balancers' server pool for the ***casdev.newschool.edu*** virtual address.</td>
        </tr>
        </tbody>
    </table>

And there are two sample client application servers:

<table>
    <colgroup>
        <col width="25%" />
        <col width="75%" />
    </colgroup>
    <tbody>
        <tr>
            <td markdown="span">***casdev-casapp***<br />192.168.100.201</td>
            <td markdown="span">A user-facing client application (Apache web server) used to test the CAS protocol and attribute release.</td>
        </tr>
        <tr>
            <td markdown="span">***casdev-samlsp***<br />192.168.100.202</td>
            <td markdown="span">A user-facing client application (Apache web server) used to test the SAML 2.0 protocol and attribute release.</td>
        </tr>
    </tbody>
</table>

The environment also includes a single virtual address on the F5 load balancers (also in the ***newschool.edu*** domain):

<table>
    <colgroup>
        <col width="25%" />
        <col width="75%" />
    </colgroup>
    <tbody>
        <tr>
            <td markdown="span">***casdev***<br />192.168.200.10</td>
            <td markdown="span">User-facing domain name and virtual address that manages access to the pool of CAS servers (***casdev-srvNN***) to provide load balancing, high availability, and fault tolerance.</td>
        </tr>
    </tbody>
</table>

Each of the six development servers is a VMware virtual machine running Red Hat Enterprise Linux (RHEL) 7 (64-bit) on 2 CPUs with 4 GB of RAM and 20 GB of disk space, which are the minimums recommended in the CAS 5 documentation.

## References

* [CAS 5: Installation Requirements][casdoc-inst-req]

{% include reflinks.md %}
