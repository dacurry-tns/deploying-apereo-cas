---
title: Update the CAS client configuration
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_ldap_resolution-release_update-the-cas-client-config.html
summary:
---

Now that the CAS server has been configured to resolve attributes and release them to the CAS client, the CAS client has to be configured to ask for them.

## Update `mod_auth_cas` settings

Edit the file `/etc/httpd/conf.d/cas.conf` on the client server (***casdev-casapp*** ) and make the following changes:

1. In the `<Directory>` directive, add a line to set `CASAuthNHeader` to `On`.
2. At the bottom of the file, change the value of the `CASValidateURL` setting from `.../serviceValidate` to `.../samlValidate`.
3. At the bottom of the file, add a line to set `CASValidateSAML` to `On`.

The result should look like this:

```apache
LoadModule auth_cas_module modules/mod_auth_cas.so

<Directory "/var/www/html/secured-by-cas">
    <IfModule mod_auth_cas.c>
        AuthType        CAS
        CASAuthNHeader  On
    </IfModule>

    Require valid-user
</Directory>

<IfModule mod_auth_cas.c>
    CASLoginUrl           https://casdev.newschool.edu/cas/login
    CASValidateUrl        https://casdev.newschool.edu/cas/samlValidate
    CASCookiePath         /var/cache/httpd/mod_auth_cas/
    CASValidateSAML       On
    CASSSOEnabled         On
    CASDebug              Off
</IfModule>
```

## Restart HTTPD

Run the command

```console
casdev-casapp# systemctl restart httpd
```

to restart the HTTPD server with the new configuration. Check the log files in `/var/log/httpd` for errors.
