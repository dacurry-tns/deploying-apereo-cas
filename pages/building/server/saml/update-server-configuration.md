---
title: Update the server configuration
last_updated: October 27, 2017
sidebar: main_sidebar
permalink: building_server_saml_update-server-configuration.html
summary:
---

The CAS server's IdP functionality requires some adjustments to Tomcat's settings, a couple of new CAS property settings, and the creation of a cache directory where SAML2 metadata can be stored.

## Adjust Tomcat settings

The SAML2 protocol requires the use of large (2MB) HTTP header sets and large (2MB) HTTP POST payloads. To enable this support on Tomcat's HTTP connector, edit the file `/opt/tomcat/latest/conf/server.xml` on the master build server (***casdev-master***) and locate the definition of the HTTPS connector (around line 89), which should look something like this:

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

Add the `maxHttpHeaderSize` and `maxPostSize` attributes to the connector definition, like this:

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
    sslImplementationName="org.apache.tomcat.util.net.openssl.OpenSSLImplementation"
    SSLEnabled="true" maxHttpHeaderSize="2097152" maxPostSize="2097152"
    connectionTimeout="20000" maxThreads="150">
```

## Configure SAML IdP properties

Add the following settings to `etc/cas/config/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***) to configure the SAML IdP:

```properties
cas.authn.samlIdp.entityId:             ${cas.server.prefix}/idp
cas.authn.samlIdp.scope:                newschool.edu
```

The `entityId` parameter is the URL by which the IdP is known to clients (SPs). The `scope` parameter identifies the "scope" in which attributes returned by the IdP apply; this is typically a DNS domain.

## Create the metadata cache directory

Create the directory `etc/cas/saml` in the `cas-overlay-template` directory on the master build server:

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# mkdir etc/cas/saml
```

## Adjust the server installation script

If you [created an installation shell script][building_server_install-and-test-the-cas-application] earlier, edit that script and, just after the line that extracts the `tar` file (around line 12), add a `chmod` command to restore group write permission to the `etc/cas/saml` directory so the CAS server can create files there:

```shell
cd /
rm -rf etc/cas/config etc/cas/services
tar xzf /tmp/cassrv-files.tgz etc/cas
chmod g+w etc/cas/saml
```

## References

* [CAS 5: Configuration Properties: SAML IdP][casdoc-saml-idp-props]

{% include reflinks.md %}
{% include links.html %}
