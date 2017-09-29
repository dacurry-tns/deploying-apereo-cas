---
title: Update the CAS client configuration
last_updated: September 29, 2017
sidebar: main_sidebar
permalink: building_server_mfa_update-the-cas-client-config.html
summary:
---

To allow the CAS client to test/demonstrate both content secured only by CAS and content secured by both CAS and Duo at the same time, create a new secure content area on the CAS client server and configure Apache HTTPD to protect it.

## Create a new secure content area

Make a copy of the existing secure content area on the client server (***casdev-casapp***):

```console
casdev-casapp# cd /var/www/html
casdev-casapp# cp -rp secured-by-cas secured-by-cas-duo
```

Then edit the file `secured-by-cas-duo/index.html` and update the paragraph of text to reflect the requirements to view it:

```html
<p><big>This is some secure content. You should not be able to see it
 until you have entered your username and password and authenticated
 with Duo.</big></p>
```

Leave the rest of the file unchanged.

## Update `mod_auth_cas` settings

Edit the file `/etc/httpd/conf.d/cas.conf` on the client server (***casdev-casapp***) and create another `<Directory>` element for the new secure content area created above:

```apache
<Directory "/var/www/html/secured-by-cas-duo">
    <IfModule mod_auth_cas.c>
        AuthType        CAS
        CASAuthNHeader  On
    </IfModule>

    Require valid-user
</Directory>
```

Except for the path name of the directory, it should be identical to the other `<Directory>` elements.

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
        <p><big>Click <a href="secured-by-cas-duo/index.php">here</a> for some
          content secured by username/password and Duo MFA.</big></p>
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
