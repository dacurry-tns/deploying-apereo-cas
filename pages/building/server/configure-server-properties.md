---
title: Configure server properties
last_updated: September 26, 2017
sidebar: main_sidebar
permalink: building_server_configure-server-properties.html
summary:
---

By default, CAS expects to find its configuration files in the operating system directory `/etc/cas`. Almost every aspect of CAS server configuration is controlled via settings stored in the `cas.properties` file located in the `/etc/cas/config` directory. The Maven WAR overlay template provides a "source" for this file (which makes it easy to manage with Git).

## Configure server name information

There are three properties that provide naming information to the CAS server:

<table>
    <colgroup>
        <col width="25%" />
        <col width="75%" />
    </colgroup>
    <tbody>
        <tr>
            <td markdown="span">`cas.server.name`</td>
            <td markdown="span">The top-level URL (protocol, domain name, and port) of the web/application server running the CAS server.</td>
        </tr>
        <tr>
            <td markdown="span">`cas.server.prefix`</td>
            <td markdown="span">The URL of the CAS web application on the web/application server. This string gets prepended to the various CAS-specific URLs used by the server.</td>
        </tr>
        <tr>
            <td markdown="span">`cas.host.name`</td>
            <td markdown="span">The name of the CAS host to be appended to ticket IDs. This value is normally determined automatically, but can be explicitly set in cases where that value may be incorrect (e.g., when hosting CAS servers for multiple domains on the same host).</td>
        </tr>
    </tbody>
</table>

Edit the file `etc/cas/config/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***) and locate the lines for `cas.server.name` and `cas.server.prefix` properties at the top of the file. Set `cas.server.name` to the correct value by replacing `cas.example.org` with the host name attached to the virtual address on the load balancer's virtual interface and removing the port part of the URL (since we're running on the standard SSL/TLS port). Then, rather than duplicating that information for `cas.server.prefix`, use variable substitution to incorporate the value of `cas.server.name`:

```properties
cas.server.name:                https://casdev.newschool.edu
cas.server.prefix:              ${cas.server.name}/cas
```

Since we will (eventually) have multiple servers generating tickets, we want to leave `cas.host.name` unset (the default). This will result in each ticket having a ticket ID that includes the host name of the server that actually created the ticket, which will make it easier to debug ticket issues. If we were to set `cas.host.name`, all the tickets would have the same "host name" in their ticket IDs, and it would be impossible to tell which server actually created the ticket.

## Configure ticket granting cookie encryption

The CAS server uses a ticket granting cookie in the browser to maintain login state during single sign-on sessions. A client can present this cookie to CAS in lieu of primary credentials and, provided it is valid, will be authenticated. The contents of the cookie should be encrypted to protect them, and when running in a multi-node environment, all of the nodes must use the same keys. Add the following lines to `etc/cas/config/cas.properties`:

```properties
cas.tgc.secure:                 true
cas.tgc.signingKey:
cas.tgc.encryptionKey:
```

Now visit the [JSON Web Key Generator][json-web-key-gen] and click on the "Shared Secret" tab. Enter `512` into the "Key Size" field, select `HS256` from the "Algorithm" drop-down, and click the "New Key" button. Copy the value of the `k` parameter from the "Key" dialog box and enter it as the value for the `cas.tgc.signingKey` property.

Then enter `256` into the "Key Size" field, select `HS256` from the "Algorithm" drop-down, and click "New Key" again, and enter that value for the `cas.tgc.encryptionKey` property. When finished, you should have something like this:

```properties
cas.tgc.secure:                 true
cas.tgc.signingKey:             bMpP_eHgIsL1kz_cnxEqYo9Bb384V70eZIvWctQ5V6xTO4P6wsQjFlglD9OSQNlFdb0mT2Q1E3qXdo05_tzrjQ
cas.tgc.encryptionKey:          r88iOMdbRMLOkITV54kax4WgadTdzUYSBXNhOp_oqS0
```

## Configure Spring Webflow encryption

CAS uses Spring Webflow to manage the authentication sequence, and this also needs to be encrypted. Add the following lines to `etc/cas/config/cas.properties`:

```properties
cas.webflow.signing.key:
cas.webflow.encryption.key:
```

Using the [JSON Web Key Generator][json-web-key-gen] again (see above), generate an `HS256` key of size `512` and enter it for the value of the `cas.webflow.signing.key` property. Generate another `HS256` key, of size `96`, and enter it for the value of the `cas.webflow.encryption.key` property. When finished, you should have something like this:

```properties
cas.webflow.signing.key:          hGapVlP6pCzIUo_CCboRszQpvWFPazmyuWsBUOoWYqUQqMKw55al5c_EGH6VBtjpIVUqEAXcvLQjQ8HaVBEmDw
cas.webflow.encryption.key:       nLMVSwhtFDIQKvBa
```

{% include tip.html content="The online JSON Web Key Generator is provided by the Mitre Corporation and the MIT Kerberos and Internet Trust Consortium, and is simply a web-based interface to the [json-web-key-generator][json-web-key-gen-git] project, also provided by Mitre/MIT. The project can be cloned from GitHub and built locally if you don't trust the online generator, or you can download and use a pre-built copy from the CAS project by running the command<br/><br/>
<code># curl -LO https://raw.githubusercontent.com/apereo/cas/master/etc/jwk-gen.jar</code><br/><br/>
Keys can then be generated using the command<br/><br/>
<code># java -jar jwk-gen.jar -t oct -s [size]</code>" %}

## References

* [CAS 5: SSO Session Cookie][casdoc-sso-cookie]
* [CAS 5: Webflow Session][casdoc-webflow-sess]

{% include reflinks.md %}
