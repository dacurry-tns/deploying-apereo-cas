---
title: Configure HTTPD to use CAS
last_updated: October 27, 2017
sidebar: main_sidebar
permalink: building_casclient_configure-httpd-to-use-cas.html
summary:
---

Now that the `mod_auth_cas` plugin has been built and installed, it can be configured, and some web content can be created to secure with it.

{% include note.html content="The steps in this section should be performed on the client server (***casdev-casapp***), not the master build server (***casdev-master***)." %}

## Configure `mod_auth_cas` settings

Create the file `/etc/httpd/conf.d/cas.conf` with the following contents to configure the `mod_auth_cas` module:

```apache
LoadModule auth_cas_module modules/mod_auth_cas.so

<Directory "/var/www/html/secured-by-cas">
    <IfModule mod_auth_cas.c>
        AuthType CAS
    </IfModule>

    Require valid-user
</Directory>

<IfModule mod_auth_cas.c>
    CASLoginUrl           https://casdev.newschool.edu/cas/login
    CASValidateUrl        https://casdev.newschool.edu/cas/serviceValidate
    CASCookiePath         /var/cache/httpd/mod_auth_cas/
    CASSSOEnabled         On
    CASDebug              Off
</IfModule>
```

If the CAS server is using a self-signed TLS/SSL certificate, the following line will also be needed:

```apache
    CASCertificatePath    /etc/pki/tls/certs/casdev.crt
```

and a copy of the public certificate should be installed in `/etc/pki/tls/certs/casdev.crt`.

## Create the cookie cache directory

Run the commands

```console
casdev-casapp# mkdir /var/cache/httpd/mod_auth_cas
casdev-casapp# chown apache.apache /var/cache/httpd/mod_auth_cas
casdev-casapp# chmod 700 /var/cache/httpd/mod_auth_cas
```

to create the directory specified in the `CASCookiePath` directive above.

## Restart HTTPD

Run the command

```console
casdev-casapp# systemctl restart httpd
```

to restart the HTTPD server with the new configuration. Check the log files in `/var/log/httpd` for errors.

## Create example content

Edit the file `/var/www/html/index.php` and replace the call to `phpinfo()` with a link to another file, like this:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Hello, World!</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  </head>
  <body>
    <div class="container">
      <h1>Hello, World!</h1>
        <p><big>The quick brown fox jumped over the lazy dogs.</big></p>
        <p><big>Click <a href="secured-by-cas/index.php">here</a> for some secure content.</big></p>
    </div>
  </body>
</html>
```

Then create a directory, `/var/www/html/secured-by-cas`, and create the file `/var/www/html/secured-by-cas/index.php` with the following contents:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Hello, World!</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  </head>
  <body>
    <div class="container">
      <h1>Secured Content</h1>
      <p><big>This is some secure content. You should not be able to see it until you have entered your username
        and password.</big></p>
      <h2>Attributes Returned by CAS</h2>
      <?php
        echo "<pre>";

        if (array_key_exists('REMOTE_USER', $_SERVER)) {
            echo "REMOTE_USER = " . $_SERVER['REMOTE_USER'] . "<br>";
        }

        $headers = getallheaders();
        foreach ($headers as $key => $value) {
            if (strpos($key, 'CAS_') === 0) {
                echo substr($key, 4) . " = " . $value . "<br>";
            }
        }

        echo "</pre>";
      ?>
    </div>
  </body>
</html>
```

The PHP code here will display environment variables and HTTP headers that are used by `mod_auth_cas` to pass attributes returned by the CAS server along to the web application.
