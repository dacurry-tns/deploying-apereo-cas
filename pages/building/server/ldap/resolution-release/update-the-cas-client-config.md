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

1. In the `<Directory>` directive, add a line to set `CASAuthNHeader` to `On`. This tells `mod_auth_cas` to add an HTTP header containing the user returned by CAS.
2. Add a second `<Directory>` directive, just like the first, except using the path `/var/www/html/return-mapped`
2. At the bottom of the file, change the value of the `CASValidateURL` setting from `.../serviceValidate` to `.../samlValidate`. This is the endpoint provided by the server for authentcating users and returning attribues via SAML 1.1.
3. At the bottom of the file, add a line to set `CASValidateSAML` to `On`. This tells `mod_auth_cas` to use SAML 1.1 to retrieve user attributes and store them as HTTP headers.

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

<Directory "/var/www/html/return-mapped">
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

## Create a new secure content area

Make a copy of the existing secure content area on the client server (***casdev-casapp***):

```console
casdev-casapp# cd /var/www/html
casdev-casapp# cp -rp secured-by-cas return-mapped
```

Then edit the file `return-mapped/index.html` and update the heading and paragraph of text to reflect the requirements to view it:

```html
<h1>Return Mapped Attributes</h1>
<p><big>This is some secure content. You should not be able to see it
 until you have entered your username and password. The attributes in
 the list below should have their "new" names as a result of using a
 "Return Mapped" attribute release policy.</big></p>
```

Leave the rest of the file unchanged.

## Update the public content page

Update `/var/www/html/index.php` to include a link to the new secure area:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Hello, World!</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet"
      href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  </head>
  <body>
    <div class="container">
      <h1>Hello, World!</h1>
        <p><big>The quick brown fox jumped over the lazy dogs.</big></p>
        <p><big>Click <a href="secured-by-cas/index.php">here</a> for some
          content secured by username and password.</big></p>
        <p><big>Click <a href="return-mapped/index.php">here</a> to see the
          results of the "Return Mapped" attribute release policy.</big></p>
    </div>
  </body>
</html>
```

## Restart HTTPD

Run the command

```console
casdev-casapp# systemctl restart httpd
```

to restart the HTTPD server with the new configuration. Check the log files in `/var/log/httpd` for errors.

## References

* [GitHub repo for `mod_auth_cas`][mod_auth_cas]

{% include reflinks.md %}
