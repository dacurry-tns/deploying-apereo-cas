---
title: Configure Luminis LDAP authentication properties
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_ldap_authentication_config-luminis-auth-properties.html
summary:
---

CAS allows multiple LDAP directories to be queried when authenticating users. Because our Active Directory does not include all our users, we also need to authenticate against the LDAP server included with Ellucian's Luminis user portal.

Add the following settings to `etc/cas/config/cas.properties`, below the Active Directory settings added in the previous section, to authenticate against Luminis LDAP:

```properties
cas.authn.ldap[1].order:                1
cas.authn.ldap[1].name:                 Luminis LDAP
cas.authn.ldap[1].type:                 AUTHENTICATED
cas.authn.ldap[1].ldapUrl:              ldaps://janus.newschool.edu
cas.authn.ldap[1].useSsl:               true
cas.authn.ldap[1].subtreeSearch:        true
cas.authn.ldap[1].userFilter:           uid={user}
cas.authn.ldap[1].baseDn:               ou=People,o=cp
cas.authn.ldap[1].bindDn:               uid=ldap_ssotest,ou=People,o=cp
cas.authn.ldap[1].bindCredential:       xxxxxxxxxxxx
cas.authn.ldap[1].principalAtrributeId: uid
```

The `[1]` in the property names indicates that this is the second LDAP source to be configured (Active Directory was `[0]`).

The properties used above are:

<table>
    <colgroup>
        <col width="25%" />
        <col width="75%" />
    </colgroup>
    <tbody>
        <tr>
            <td markdown="span">`order`</td>
            <td markdown="span">When multiple authentication sources are configured, the CAS server looks for the user in one source after another until the user is found, and then the authentication is performed against that source (where it either succeeds or fails). This property influences the order in which the source is evaluated (if not specified, sources are evaluated in the order they are defined).</td>
        </tr>
        <tr>
           <td markdown="span">`name`</td>
           <td markdown="span">The name of the source. This is used when writing log file messages.</td>
        </tr>
        <tr>
            <td markdown="span">`type`</td>
            <td markdown="span">The type of authenticator to use. This should be `AUTHENTICATED` to specify the traditional "bind account" method of authentication.</td>
        </tr>
        <tr>
            <td markdown="span">`ldapUrl`</td>
            <td markdown="span">The URL of the LDAP server. In our case, we use the URL of the virtual host on the F5 load balancer, which has multiple LDAP servers behind it.</td>
        </tr>
        <tr>
            <td markdown="span">`useSsl`</td>
            <td markdown="span">Whether to use TLS/SSL when communicating with LDAP. Since we specified `ldaps` in the `ldapUrl`, this should be `true`.</td>
        </tr>
        <tr>
            <td markdown="span">`subtreeSearch`</td>
            <td markdown="span">Whether to search the entire subtree of the directory rooted at the `baseDN`. Usually this should be `true`.</td>
        </tr>
        <tr>
            <td markdown="span">`userFilter`</td>
            <td markdown="span">The LDAP filter to select the user from the directory. Luminis LDAP searches on the `uid` attribute, which is actually the user's username. The `{user}` pattern will be replaced with the username string entered by the user.</td>
        </tr>
        <tr>
            <td markdown="span">`baseDn`</td>
            <td markdown="span">The base DN to search against when retrieving attributes.</td>
        </tr>
        <tr>
            <td markdown="span">`bindDN`</td>
            <td markdown="span">The DN of the account to bind to the directory with. This account must have search privileges on the directory.</td>
        </tr>
        <tr>
            <td markdown="span">`bindCredential`</td>
            <td markdown="span">The password to the bind account.</td>
        </tr>
        <tr>
            <td markdown="span">`principalAttributeId`</td>
            <td markdown="span">The attribute to be used as the security principal identifier. In our case, `uid` is the username, which is what we want.</td>
        </tr>
    </tbody>
</table>

## Install and test on the master build server

Adding the Luminis LDAP server only required changing `cas.properties`, so there is no need to rebuild or reinstall the server. Instead, just copy the new file into place on the master build server (***casdev-master***) and restart Tomcat by running the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# cp etc/cas/config/cas.properties /etc/cas/config/cas.properties
casdev-master# systemctl restart tomcat
```

Review the contents of the log files (`/var/log/tomcat/catalina.yyyy-mm-dd.out` and `/var/log/cas/cas.log`) for errors.

Once everything has started, open up a web browser and enter the URL of the CAS application on the master build server (`https://casdev-master.newschool.edu:8443/cas/login`), and try to log in using a Luminis LDAP username and password (one that isn't also in Active Directory). The "Log In Successful" page should appear. If it doesn't, consult `/var/log/cas/cas.log` for errors. Then try logging in with an Active Directory username and password to confirm that the addition of LDAP didn't break anything.

It may help to enable debugging on the LDAP module by changing the `org.ldaptive` logging level to `debug` around line 95 in `/etc/cas/config/log4j2.xml`:

```xml
<AsyncLogger name="org.ldaptive" level="debug" />
```

and restarting Tomcat.

## Install and test on the CAS servers

Once everything is running correctly on the master build server, it can be copied to the CAS servers:

```console
casdev-master# for host in srv01 srv02 srv03
> do
> scp etc/cas/config/cas.properties casdev-${host}:/etc/cas/config/cas.properties
> ssh casdev-${host} systemctl restart tomcat
> done
casdev-master#  
```

and tested using the URL of the load balancer's virtual interface (`https://casdev.newschool.edu/cas/login`).

## Commit changes to Git

Before moving on to the next phase of configuration, commit the changes made to `pom.xml` and `cas.properties` to Git:

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add etc/cas/config/cas.properties
casdev-master# git commit -m "Added Luminis LDAP authentication"
[newschool-casdev 912da34] Added Luminis LDAP authentication
 1 file changed, 15 insertions(+)
casdev-master#  
```

## References

* [CAS 5: Configuration Properties: LDAP Authentication][casdoc-ldap-auth-props]

{% include reflinks.md %}
