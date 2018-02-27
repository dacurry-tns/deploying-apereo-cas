---
title: Configure MongoDB to use TLS/SSL
last_updated: February 27, 2018
sidebar: main_sidebar
permalink: high-avail_mongodb_configure-mongodb-to-use-tls-ssl.html
summary:
---

Communications between the CAS server and calling applications (clients) are protected by TLS/SSL to prevent the disclosure of users’ security credentials and/or CAS ticket-granting tickets. Since MongoDB will be used to store ticket-granting tickets (and other sensitive information), communications between the CAS server and MongoDB should likewise be protected by TLS/SSL.

## Generate private keys and certificate signing requests

Each of the replica set members (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) will need its own TLS/SSL certificate. Run the commands

```console
casdev-srv01# cd /etc/pki/tls/private
casdev-srv01# openssl req -nodes -newkey rsa:2048 -sha256 -keyout casdev-srv01.key -out casdev-srv01.csr
Generating a 2048 bit RSA private key
................................................................................
....................+++
................................................................................
...............+++
writing new private key to 'casdev-srv01.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []:New York
Locality Name (eg, city) [Default City]:New York
Organization Name (eg, company) [Default Company Ltd]:The New School
Organizational Unit Name (eg, section) []:IT
Common Name (eg, your name or your server's hostname) []:casdev-srv01.newschool.edu
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
casdev-srv01#
```

on ***casdev-srv01*** to generate a private key and certificate signing request. (Replace the contents of the Distinguished Name fields with values appropriate for your organization.) Repeat the commands on ***casdev-srv02*** and ***casdev-srv03*** to generate private keys and certificate signing requests on those servers as well, making the obvious host name substitutions on each server.

Submit all three certificate signing requests (`casdev-srvNN.csr`) to your certificate authority to obtain certificates. When the certificates come back from the certificate authority, copy the certificate for each server and any intermediate certificate(s) into `/etc/pki/tls/certs` on the applicable server, saving them as `casdev-srvNN.crt`, `casdev-srvNN-intermediate.crt`, `casdev-srvNN-root.crt`, etc. (you may need to separately download the root certificate from the certificate authority's web site). If your certificate authority offers multiple certificate formats, opt for the PEM format, which looks like:

```
-----BEGIN CERTIFICATE-----
AQEFAAOCAQ8AMIIBCgKCAQEAtGCKiysqhQF4/AA5Pvi7EIIRqbtVx/IF0CAFK8lv
6uDJDHjd7bSNhhzYJxUNCdN0DacYT5wI/s4n3mLEXQrIt0KsUdPD+s7qP9Lw05hI
WaG7KhP6RZ+UtWSvHwIZJUHvlJvh2GlARw/XwV3iHG3mxfl5nCLNihAR9S1r2qEY
...several more lines of base64-encoded data...
-----END CERTIFICATE----
```

## Combine the certificate and private key into a single `.pem` file

The MongoDB server requires the certificate and the private key to be stored in a single file. Run the commands

```console
casdev-srv01# cd /etc/pki/tls
casdev-srv01# cat private/casdev-srv01.key certs/casdev-srv01.crt > /var/lib/mongo/mongod-cert.pem
casdev-srv01# chown mongod.mongod /var/lib/mongo/mongod-cert.pem
casdev-srv01# chmod 400 /var/lib/mongo/mongod-cert.pem
```

on ***casdev-srv01*** to create the combined file in `/var/lib/mongo/mongod-cert.pem`. Repeat these commands on ***casdev-srv02*** and ***casdev-srv03***.

## Combine the root and intermediate certificates into a single `.pem` file

The MongoDB server also requires that the certificate chain (the intermediate certificate(s) and the root certificate) from the certificate authority be provided in a single file. Run the commands

```console
casdev-srv01# cd /etc/pki/tls
casdev-srv01# cat certs/casdev-srv01-intermediate.crt casdev-srv01-root.crt > /var/lib/mongo/mongod-cafile.pem
casdev-srv01# chown mongod.mongod /var/lib/mongo/mongod-cafile.pem
casdev-srv01# chmod 400 /var/lib/mongo/mongod-cafile.pem
```

on ***casdev-srv01*** to create the combined file in `/var/lib/mongo/mongod-cafile.pem`. Repeat these commands on ***casdev-srv02*** and ***casdev-srv03***.

## Update the MongoDB configuration file

Edit the `/etc/mongod.conf` file on ***casdev-master*** and add an `ssl` subsection to the `net` section, as shown below:

```yaml
net:
  port: 27017
  bindIp: 0.0.0.0
  ssl:
    mode: requireSSL
    allowConnectionsWithoutCertificates: true
    PEMKeyFile: /var/lib/mongo/mongod-cert.pem
    CAFile: /var/lib/mongo/mongod-cafile.pem
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

## Test the TLS/SSL configuration

To test the TLS/SSL configuration, use the `mongo` shell with the `--ssl` option to verify that you can connect to the server and execute a command. Run the commands

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">casdev-master# </span><span class="nc">mongo</span><span class="kv"> -u mongoadmin -p --authenticationDatabase admin --ssl --host rs0/casdev-srv01.newschool.edu,casdev-srv02.newschool.edu,casdev-srv03.newschool.edu
</span>MongoDB shell version v3.6.0
<span class="ni">Enter password:</span>
connecting to: mongodb://casdev-srv01.newschool.edu:27017,casdev-srv02.newschool.edu:27017,casdev-srv03.newschool.edu:27017/?replicaSet=rs0
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Starting new replica set monitor for rs0/casdev-srv01.newschool.edu:27017,casdev-srv02.newschool.edu:27017,casdev-srv03.newschool.edu:27017
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Successfully connected to casdev-srv02.newschool.edu:27017 (1 connections now open to casdev-srv02.newschool.edu:27017 with a 5 second timeout)
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [ReplicaSetMonitor-TaskExecutor-0] Successfully connected to casdev-srv01.newschool.edu:27017 (1 connections now open to casdev-srv01.newschool.edu:27017 with a 5 second timeout)
YYYY-MM-DDTHH:MM:SS.sss-0000 I NETWORK  [thread1] Successfully connected to casdev-srv03.newschool.edu:27017 (1 connections now open to casdev-srv03.newschool.edu:27017 with a 5 second timeout)
MongoDB server version: 3.6.0
<span class="ni">rs0:PRIMARY&gt; </span><span class="nc">show</span><span class="kv"> dbs</span>
admin   0.000GB
casdb   0.000GB
config  0.000GB
local   0.014GB
<span class="ni">rs0:PRIMARY&gt; </span><span class="nc">exit</span>
bye
<span class="ni">casdev-master# </span><span class="kv">
</span></code></pre>
</div>

Note that, now that a replica set has been established, the `--host` option must be provided, listing the replica set name and the names of all the replica set members, rather than just letting `mongo` connect to the local host. This will ensure that the shell connects to the primary replica set member, regardless of which member that happens to be at the moment.

## References

* [MongoDB: Configure mongod and mongos for TLS/SSL][mongodb-ssl]

{% include reflinks.md %}
{% include links.html %}
