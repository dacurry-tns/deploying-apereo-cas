---
title: Configure Active Directory authentication properties
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_ldap_authentication_config-ad-auth-properties.html
summary:
---

Although CAS offers several dozen properties for controlling how LDAP authentication is performed, most of them come with reasonable defaults and do not have to be configured in normal circumstances. The complete list of properties can be found in the CAS documentation.

Add the following settings to `etc/cas/config/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***) to authenticate against Active Directory:

```properties
cas.authn.ldap[0].order:                0
cas.authn.ldap[0].name:                 Active Directory
cas.authn.ldap[0].type:                 AD
cas.authn.ldap[0].ldapUrl:              ldaps://zuul.newschool.edu
cas.authn.ldap[0].useSsl:               true
cas.authn.ldap[0].userFilter:           sAMAccountName={user}
cas.authn.ldap[0].baseDn:               ou=TNSUsers,dc=tns,dc=newschool,dc=edu
cas.authn.ldap[0].dnFormat:             cn=%s,ou=TNSUsers,dc=tns,dc=newschool,dc=edu
```

The `[0]` in the property names indicates that this is the first LDAP source to be configured. Additional sources will be `[1]`, `[2]`, etc. (more on this in [Configure Luminis LDAP authentication properties][building_server_ldap_authentication_config-luminis-auth-properties]).

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
            <td markdown="span">The type of authenticator to use. This should be `AD` for Active Directory.</td>
        </tr>
        <tr>
            <td markdown="span">`ldapUrl`</td>
            <td markdown="span">The URL of the Active Directory server. In our case, we use the URL of the virtual host on the F5 load balancer, which has multiple Active Directory servers behind it.</td>
        </tr>
        <tr>
            <td markdown="span">`useSsl`</td>
            <td markdown="span">Whether to use TLS/SSL when communicating with Active Directory. Since we specified `ldaps` in the `ldapUrl`, this should be `true`.</td>
        </tr>
        <tr>
            <td markdown="span">`userFilter`</td>
            <td markdown="span">The LDAP filter to select the user from the directory. Active Directory typically searches on the `sAMAccountName` attribute. The `{user}` pattern will be replaced with the username string entered by the user.</td>
        </tr>
        <tr>
            <td markdown="span">`baseDn`</td>
            <td markdown="span">The base DN to search against when retrieving attributes. The "usual" value for this is more like `ou=Users,dc=example,dc=org`, but for historical reasons we keep our users in a different OU.</td>
        </tr>
        <tr>
            <td markdown="span">`dnFormat`</td>
            <td markdown="span">A format string to generate the user DN to be authenticated. In the string, `%s` will be replaced with the username entered on the login form. The "usual" value of this string is something more like `uid=%s,ou=Users,dc=example,dc=org`, but we do not use the `uid` attribute in our Active Directory schema, we use `cn` instead.</td>
        </tr>
    </tbody>
</table>

## Install and test on the master build server

Use the scripts [created earlier][building_server_install-and-test-the-cas-application] (or repeat the commands) to install the updated CAS application and configuration files on the master build server (***casdev-master***):

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# sh /opt/scripts/cassrv-install.sh
---Installing on casdev-master.newschool.edu
Installation complete.
casdev-master#  
```

Review the contents of the log files (`/var/log/tomcat/catalina.yyyy-mm-dd.out` and `/var/log/cas/cas.log`) for errors.

Once everything has started, open up a web browser and enter the URL of the CAS application on the master build server (`https://casdev-master.newschool.edu:8443/cas/login`), and try to log in using an Active Directory username and password. The "Log In Successful" page should appear. If it doesn't, consult `/var/log/cas/cas.log` for errors.

It may help to enable debugging on the LDAP module by changing the `org.ldaptive` logging level to `debug` around line 95 in `/etc/cas/config/log4j2.xml`:

```xml
<AsyncLogger name="org.ldaptive" level="debug" />
```

and restarting Tomcat.

## Install and test on the CAS servers

Once everything is running correctly on the master build server, it can be copied to the CAS servers using the scripts [created earlier][building_server_install-and-test-the-cas-application]:

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# for host in srv01 srv02 srv03
> do
> scp -p /tmp/cassrv-files.tgz casdev-${host}:/tmp/cassrv-files.tgz
> scp -p /opt/scripts/cassrv-install.sh casdev-${host}:/tmp/cassrv-install.sh
> ssh casdev-${host} sh /tmp/cassrv-install.sh
> done
casdev-master#  
```

and tested using the URL of the load balancer's virtual interface (`https://casdev.newschool.edu/cas/login`).

## Commit changes to Git

Before moving on to the next phase of configuration, commit the changes made to `pom.xml` and `cas.properties` to Git:

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# git add etc/cas/config/cas.properties
casdev-master# git commit -m "Added Active Directory authentication"
[newschool-casdev 584aa7c] Added Active Directory authentication
 1 file changed, 16 insertions(+)
casdev-master#  
```

## References

* [CAS 5: Configuration Properties: LDAP Authentication][casdoc-ldap-auth-props]

{% include reflinks.md %}
{% include links.html %}
