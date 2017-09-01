---
title: Configure TLS/SSL settings
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_configure-tlsssl-settings.html
summary:
---

All of the servers in the load balancer server pool (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) will use the same TLS/SSL certificate (because they will all identify themselves by the same host name), so only one certificate needs to be created.

All of the steps in this section should be performed on the master build server (***casdev-master***); the results will be copied to the CAS servers (***casdev-srv01***, ***casdev-srv02***, and ***casdev-srv03***) later.

## Generate a private key and certificate signing request

The private key and certificate signing request can be generated using either the `openssl` command or the Java `keytool` command. The former creates keys and certificates in standard formats that have to be imported to a Java keystore for use by Tomcat; the latter creates a Java keystore from which keys and certificates have to be exported for use by non-Java applications. There really isn't any technical reason to choose one tool over the other; use whichever one you're most comfortable with. In this document we will use `openssl` and then import to a Java keystore in the next step. Run the commands

```console
casdev-master# cd /etc/pki/tls/private
casdev-master# openssl req -nodes -newkey rsa:2048 -sha256 -keyout casdev.key -out casdev.csr
Generating a 2048 bit RSA private key
.........+++
.............+++
writing new private key to 'casdev.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []: New York
Locality Name (eg, city) [Default City]: New York
Organization Name (eg, company) [Default Company Ltd]: The New School
Organizational Unit Name (eg, section) []: IT
Common Name (eg, your name or your server's hostname) []: casdev.newschool.edu
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
casdev-master#  
```

to generate a private key and certificate signing request. (Replace the contents of the Distinguished Name fields with values appropriate for your organization.) Submit the certificate signing request (`casdev.csr`) to your certificate authority to obtain a certificate. When the certificate comes back from the certificate authority, copy it and any intermediate certificate(s) into `/etc/pki/tls/certs`, saving them as `casdev.crt`, `casdev-intermediate.crt`, etc. If your certificate authority offers multiple certificate formats, opt for the PEM format, which looks like:

```
-----BEGIN CERTIFICATE-----
AQEFAAOCAQ8AMIIBCgKCAQEAtGCKiysqhQF4/AA5Pvi7EIIRqbtVx/IF0CAFK8lv
6uDJDHjd7bSNhhzYJxUNCdN0DacYT5wI/s4n3mLEXQrIt0KsUdPD+s7qP9Lw05hI
WaG7KhP6RZ+UtWSvHwIZJUHvlJvh2GlARw/XwV3iHG3mxfl5nCLNihAR9S1r2qEY
...several more lines of base64-encoded data...
-----END CERTIFICATE----
```

## Import the certificate into a Java keystore {#importtokeystore}

To import a certificate into a Java keystore, the first step is to combine the certificate chain (the host certificate and any intermediate certificates) into a single file. If you are using a self-signed certificate or a certificate authority whose root certificate is not distributed with RHEL 7, include the root certificate as well. The certificates must be placed in certificate chain order from "lowest" to "highest" as shown:

```console
casdev-master# cd /etc/pki/tls/certs
casdev-master# cat casdev.crt casdev-intermediate.crt [root.crt] > /opt/tomcat/casdev-all.crt
```

Next, create a PKCS#12-format file from the combined certificates and the private key file:

```console
casdev-master# cd /opt/tomcat
casdev-master# openssl pkcs12 -export -inkey /etc/pki/tls/private/casdev.key -in casdev-all.crt -name tomcat -out casdev.p12
Enter Export Password: changeit
Verifying - Enter Export Password: changeit
casdev-master#  
```

Be sure to enter a non-blank password, or the import command (next) will fail. Next, import the PKCS#12-format file to a Java keystore:

```console
casdev-master# keytool -importkeystore -srckeystore casdev.p12 -srcstoretype pkcs12 -destkeystore keystore.jks
Enter destination keystore password: changeit
Re-enter new password: changeit
Enter source keystore password: changeit
Entry for alias tomcat successfully imported.
Import command completed:  1 entries successfully imported, 0 entries failed or cancelled
casdev-master#  
```

{% include warning.html content="The default keystore password used by Tomcat is `changeit`, hence its use in the examples above (and further below). Obviously, something other than this default value should be used in a production Tomcat deployment." %}

Finally, delete the intermediate files and set the appropriate permissions on the keystore file:

```console
casdev-master# rm -f casdev-all.crt casdev.p12
casdev-master# chown root.tomcat keystore.jks
casdev-master# chmod 640 keystore.jks
```

## Configure Tomcat server settings

Edit the file `/opt/tomcat/latest/conf/server.xml` and make the changes described below to disable unneeded network connectors and to enable and properly configure the TLS/SSL connector.

### Disable the `SHUTDOWN` port

Locate the `Server` XML tag (around line 22), which should look something like this:

```xml
<Server port="8005" shutdown="SHUTDOWN">
```

We will be using `systemd` to manage Tomcat (see [Configure `systemd` to start Tomcat][setup_tomcat_configure-systemd-to-start-tomcat]), so the `SHUTDOWN` port is not needed. Change the port number to `-1` to disable it, as shown below:

```xml
<Server port="-1" shutdown="SHUTDOWN">
```

### Disable the HTTP connector

Locate the first definition of an HTTP connector on port 8080 (around line 69), which should look something like this:

```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

As discussed previously, all CAS communications should occur over a secure (TLS) channel, so this connector is not needed. Comment it out by inserting `<!--` and `-->` around it, like this:

```xml
<!--
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
-->
```

### Enable and configure the HTTPS connector {#enablehttpsconnector}

Locate the first definition of a TLS/SSL connector on port 8443 (around line 88). As shipped with Tomcat, it will be commented out and look something like this:

```xml
<!--
 <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
            maxThreads="150" SSLEnabled="true">
     <SSLHostConfig>
         <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"
                      type="RSA" />
     </SSLHostConfig>
 </Connector>
 -->
```

Remove the comment lines (`<!--` and `-->`) and change the connector definition to look like this:

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
    sslImplementationName="org.apache.tomcat.util.net.openssl.OpenSSLImplementation"
    SSLEnabled="true" connectionTimeout="20000" maxThreads="150">
    <SSLHostConfig
        ciphers="ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS"
        honorCipherOrder="true" protocols="all,-SSLv2Hello,-SSLv2,-SSLv3"
        disableSessionTickets="true">
        <Certificate
            certificateKeystoreFile="/opt/tomcat/keystore.jks"
            certificateKeystorePassword="changeit"
            type="RSA" />
    </SSLHostConfig>
    <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
</Connector>
```

To obtain the most up-to-date list of ciphers for the `ciphers` attribute, use the [Mozilla SSL Configuration Generator][mozilla-ssl-gen] and select "Apache" and "Intermediate." Then copy and paste the value of the `SSLCipherSuite` parameter.

{% include important.html content="The value of the `certificateKeystorePassword` attribute should be the same password you entered for the keystore file in [Import the certificate into a Java keystore][setup_tomcat_configure-tlsssl-settings.html#importtokeystore], above." %}

{% include note.html content="The configuration above uses TCP port 8443 for the HTTPS (TLS/SSL) port. This is the conventional port used by Tomcat and CAS, but is not a requirement. It's also possible, for example, to use the well-known HTTPS port (TCP 443) or any other port, simply by changing the value of the port attribute on the connector definition." %}

### Disable the AJP connector

Locate the definition of the AJP (Apache JServ Protocol) connector on port 8009 (around line 115), which should look something like this:

```xml
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```

AJP is mostly used when front-ending Tomcat with Apache HTTPD. We're not doing that, so this connector is not needed. Comment it out by inserting `<!--` and `-->` around it, like this:

```xml
<!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->
```

## References

* [Tomcat Configuration Reference: The HTTP Connector][tomcat-http-conn]
* [Mozilla Wiki: Security/Server Side TLS][mozilla-ssl-tls]
* [Digital Ocean: OpenSSL Essentials][digital-ocean-openssl]
* [Acmetek: Java Keytool Commands][acmetek-java-keytool]

{% include reflinks.md %}
{% include links.html %}
