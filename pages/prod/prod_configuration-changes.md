---
title: Configuration changes
last_updated: October 18, 2018
sidebar: main_sidebar
permalink: prod_configuration-changes.html
summary:
---

Moving the CAS server from the development environment to the test environment, and then from the test environment to production, requires a number of changes to the configuration settings to identify the servers in the new environment, update encryption keys, etc. There are also some other configuration changes that should be made (or at least considered), especially when moving to the production environment.

## Update CAS configuration settings

Several configuration settings created during the course of building the servers and enabling various features are specific to the environment in which the servers were built, and need to be updated when moving to another environment:

1. Several properties in `etc/cas/config/cas.proprties` must be updated with values for the new environment (in the list below, moving to the production environment is assumed):
    1. The `cas.server.name` property must be updated with the production server's domain name.
    2. If the production Active Directory/LDAP servers are not the same ones as those used by the development/test environment, the `cas.authn.ldap[n].ldapUrl` and `cas.authn.attributeRepository/ldapUrl` properties must be updated. The `bindDn` and `bindCredential` properties may also need to be updated.
    3. If the production environment uses different load balancers than the development/test environment, the `cas.adminPageSecurity.ip` regular expression must be updated to match the IP addresses of the production load balancers.
    4. The `mongo.hosts` "pseudo-property" from which the MongoDB connection string is constructed must be updated to contain the list of host names of the production MongoDB replica set members.
    5. New ticket granting cookie encryption keys (`cas.tgc.crypto.signing.key` and `cas.tgc.crypto.encryption.key`) should be generated for the production environment.
    6. New Spring Webflow encryption keys (`cas.webflow.crypto.signing.key` and `cas.webflow.crypto.encryption.key`) should be generated for the production environment.
    7. Thymeleaf caching should be enabled in the production environment by adding the `spring.thymeleaf.cache` property and setting its value to `true`.
2. Some properties in `etc/cas/config/management.properties` must also be updated:
    1. The `cas.server.name` property must be updated with the production server's domain name (***sso.newschool.edu***).
    2. The `mongo.hosts` "pseudo-property" from which the MongoDB connection string is constructed must be updated to contain the list of host names of the production MongoDB replica set members.
2. The CAS server SSL certificate in Tomcat's keystore must be [replaced][setup_tomcat_configure-tlsssl-settings] with the production SSL certificate.
3. New SAML IdP metadata, encryption keys, and signing keys should be [generated](building_server_saml_install-and-test-the-idp.html#copy-cas-generated-idp-metadata-to-the-overlay-template) for the production environment and copied to `etc/cas/saml`.
4. Additional CAS administrative users (who will have access to the management webapp and/or the dashboard) should be added to the `etc/cas/config/admusers.properties` file.
5. A new Duo protected application should be [created][building_server_mfa_configure-duo-authentication] (via the Duo administration console) for the production CAS environment and the `duoIntegrationKey`, `duoSecretKey`, and `duoApplicationKey` components of the `cas.authn.mfa.duo[n]` property in `etc/cas/config/cas.properties` updated with the new values.
6. If the production Google Apps environment is not the same as the one used by the development/test environments, the key files in `etc/cas/google` should be [regenerated][googleapps_generate-keys-and-certificates] and the new X.509 certificate uploaded to the production Google Apps environment.

Once all of the above changes have been made in the `cas-overlay-template` directory on the master build server for the environment, the [build and installation scripts][building_svcmgmt_install-and-test-the-webapp] should be used to install and test the updated CAS server and management webapp on the master build server, and then everything should be copied to the CAS servers.

## Lock down the Tomcat ROOT application

When we initially [hardened the Tomcat installation][setup_tomcat_harden-the-installation], we chose not to remove the `ROOT` web application from Tomcat's `webapps` directory because it can be useful in a development/test environment to quickly determine whether Tomcat is working properly. We even [modified the application][setup_tomcat-test-the-tomcat-installation] to display the host name, IP address, and port number of the Tomcat server. This information can still be useful in a production environment for checking on load balancer issues and the like, but from a security perspective, it's not information that should be given to the general public.

To alleviate this problem, we can add some code to the top of the `/opt/tomcat/latest/webapps/ROOT/index.jsp` (`/var/lib/tomcat/ROOT/index.jsp`) file around line 18, just below this line:

```js
<%@ page session="false" pageEncoding="UTF-8" contentType="text/html; charset=UTF-8" %>
```

This is the code to be added:

```js
<%
  String a = request.getRemoteAddr();

  if (a != null && !a.substring(0, 10).equals("192.68.50.") &&
      !a.equals("192.168.1.10") && !a.equals("192.168.1.20")) {
    // response.sendRedirect("https://casdev.newschool.edu/cas/login");
    // response.sendRedirect("https://castest.newschool.edu/cas/login");
    response.sendRedirect("https://sso.newschool.edu/cas/login");
  }
%>
```

The code checks the IP address of the client host accessing the page and, if it's not the IT department subnet or the internal interface of one of the load balancers, sends an HTTP redirect to the CAS login page. With this addition, any attempt to access `https://sso.newschool.edu` from a non-whitelisted host will be redirected to `https://casdev.newschool.edu/cas/login`.

Note that the URL in the `response.sendRedirect()` call is dependent on the environment in which this code is running.

## Set a default application

Occasionally, users will inadvertently create a browser bookmark for the CAS login page when what they (usually) mean to do is create a bookmark for the service that directed them there. Eventually, this results in the user visiting the CAS login page without having been directed there by a service. When this happens, and the user successfully authenticates, he or she will end up at a generic "login successful" page rather than at the service he or she actually wanted. Fortunately, CAS allows a different destination to be configured by adding a line to `etc/cas/config/cas.properties`:

```properties
cas.view.defaultRedirectUrl=https://my.newschool.edu
```

We have chosen to send users to *MyNewSchool*, the New School web portal&mdash;partly because it's a commonly-used application that is probably (arguably?) where they meant to go anyway, but more importantly, because it's an application where (almost) all users have an account. Sending a user to an application where he or she doesn't have an account is even more confusing that sending him or her to the generic login success page.

## Create a standard location in which to store SAML SP metadata

When we [created the service registry definition for our SAML2 test client][building_samlclient_update-the-service-registry], we provided a URL for the `metadataLocation` attribute, telling the CAS server that it could contact the SAML2 SP at that URL to dynamically download the SP's metadata. This is well and good for those SPs that support it, but at least in our experience many, if not most, SAML2-based SaaS applications do not offer dynamically-downloadable metadata. Instead, customers are expected to manually download the metadata and store it somewhere locally for their SAML2 IdP (CAS) to use.

Unfortunately, CAS does not yet support storing SAML SP metadata in a high-availability location such as the database where the ticket registry and service registry are stored. Therefore, ensuring that SAML SP metadata information is replicated across all the CAS servers in the pool must be handled manually. To make this at least a bit easier, we have chosen to maintain "master" copies of SP metadata files in the CAS Maven overlay, so that they can be distributed alongside any other updates to the CAS application or configuration files.

We have created an `etc/cas/saml/sp-metadata` subdirectory in the `cas-overlay-template` directory, and we store all third-party metadata files there. Whenever the CAS application or configuration files are updated, this subdirectory will be included in the files distributed by the [build and installation scripts][building_svcmgmt_install-and-test-the-webapp] we use for that purpose and extracted into `/etc/cas/saml/sp-metadata/` on each of the CAS servers in the pool.

{% include note.html content="While this method has worked reasonably well for us, it's by no means perfect. In practice, when a new SAML2 service is added, the system administrator responsible for doing that just places the SP metadata into `/etc/cas/saml/sp-metadata/[servicename].xml` on one of the CAS servers and manually copies it to the others with `scp`. Once everything is working, someone is supposed to copy the final `[servicename].xml` file back to the Maven overlay template so that it can be distributed properly in the future. We've forgotten this step once or twice though, so now the checklist for updating the Maven overlay (e.g., when a new version of CAS comes out) includes an item to check the servers for any new metadata files and copy them down to the overlay." %}

## Logging to Graylog

Having multiple CAS servers behind a load balancer makes tracing things through log files difficult. Even with "sticky" sessions on the load balancer, there's no way to determine in advance which server in the pool is the one to be monitoring when trying to debug something. And if the servers are busy, going back and trying to find the one thing you needed after the fact can be difficult, as well. Since we have a Graylog server in our environment that lots of other things log to, it made sense to use that to consolidate CAS logs there as well.

{% include tip.html content="Syslog, of course, is another option. But log messages in Graylog Extended Log Format (GELF) provide significantly more information, in an easier-to-parse format, than log messages in typical Syslog format. This makes searching and filtering log messages much more robust on a Graylog server than on a Syslog server." %}

### Configure Graylog to accept input from CAS

To configure Graylog to accept input from CAS:
1. Log into the Graylog server GUI and select **System > Inputs**.
2. From the "Select input" drop-down, choose `GELF UDP` and click the green **Launch new input** button.
3. Fill in some configuration settings:
    1. Under **Node**, choose the Graylog node that should host this input.
    2. Under **Title**, enter something meaningful like "Production CAS servers".
    3. Under **Port**, choose a UDP port number to listen on (the default may be okay, but we chose to use different ports for the development, test, and production pools of CAS servers).
    4. Leave all the other fields at their defaults.
4. Click the maroon **Save** button.

### Configure CAS to log to Graylog

To configure CAS to log to Graylog, edit the file `etc/cas/config/log4j2.xml` in the `cas-overlay-template` directory. First, define a socket appender by adding the following definition underneath all the `<RollingFile` appender definitions and just above the `<CasAppender>` definitions (around line 45):

```xml
<Socket name="graylog" host="graylog.newschool.edu" protocol="udp" port="12203">
    <GelfLayout compressionType="GZIP" compressionThreshold="1024">
        <KeyValuePair key="webappName" value="cas"/>
    </GelfLayout>
</Socket>
```

Make sure that the `host` attribute contains the host name (or IP address) of the Graylog server and that the `port` attribute contains the value used when the new Graylog input was defined in the previous section.

{% include warning.html content="Graylog will accept input via TCP or UDP. However, if CAS is logging to Graylog via TCP, and the Graylog server becomes unavailable for some reason, the CAS servers can potentially run out of resources because all available threads are blocked waiting to deliver log messages to Graylog. We learned this the hard way&mdash;log to Graylog (or Syslog) with UDP." %}

Next, in the `<AsyncRoot>` definition (around line 108), add a reference to the Graylog appender (the `ref` attribute should have the same value as the `name` attribute in the `<Socket>` definition above):

```xml
<AsyncRoot level="warn">
    <AppenderRef ref="casFile"/>
    <AppenderRef ref="graylog"/>
    <!--
         For deployment to an application server running as service,
         delete the casConsole appender below
    -->
    <!-- <AppenderRef ref="casConsole"/> -->
</AsyncRoot>
```

In our experience, the `<AsyncRoot>` logger (`cas.log`) is the only one that needs to be sent to Graylog. Everything that gets logged to the audit logger (`cas_audit.log`) is also logged to the root logger, and the `perfStatsLogger` (`perfStats.log`) is only logging performance statistics. On the Tomcat side,  `catalina.out` is only helpful if the CAS application itself is failing, and if that's happening in production, it's probably isolated to a single server.

### Install the modified Log4j2 configuration

Install the updated `log4j2.xml` file on all the CAS servers:

```console
casprod-master# cd /opt/workspace/cas-overlay-template
casprod-master# cat etc/cas/config/log4j2.xml > /etc/cas/config/log4j2.xml
casprod-master# cd /
casprod-master# for h in 01 02 03 04 05
> do
> tar cf - etc/cas/config/log4j2.xml | ssh casprod-srv${h} "cd /; tar xf -"
> done
casprod-master#  
```

CAS monitors `/etc/cas/config/log4j2.xml` for changes and picks them up automatically, so there is no need to restart the application.

### Configure the management webapp to log to Graylog

Since the management webapp also runs on every server in the pool, it makes sense to send its logs to Graylog as well (it can log to the same Graylog input as the associated CAS servers). Repeat the steps above to add the Graylog `<Socket>` definition and the Graylog `<AppenderRef`> to `etc/cas/config/log4j2-management.xml` in the `cas-management-overlay` directory, and then install the modified configuration file on all the CAS servers.

## Use an Extended Validation (EV) SSL certificate

To help users identify phishing scams that make use of a forged New School SSO login page, we purchased an Extendned Validation (EV) SSL certificate for our production CAS servers, which use the ***sso.newschool.edu*** host name. Because all the most popular browsers (except Safari) display the organization name associated with the certificate in the URL bar when an EV certificate is used, it's easy to instruct users to look for this to confirm that they're accessing the "official" login page.

{% include image.html file="prod/fig27-browser-ev-certs.png" alt="Browser Screen Shot" caption="Figure 27. How browsers display EV SSL certificates" %}

{% include reflinks.md %}
{% include links.html %}
