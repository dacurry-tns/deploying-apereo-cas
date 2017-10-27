---
title: Configure HTTPD to use the SP
last_updated: October 27, 2017
sidebar: main_sidebar
permalink: building_samlclient_configure-httpd-to-use-the-sp.html
summary:
---

Now that the Shibboleth SP has been installed, the `mod_shib` Apache HTTPD module can be configured and some web content can be created to secure with it.

{% include note.html content="The steps in this section should be performed on the client server (***casdev-samlsp***), not the master build server (***casdev-master***)." %}

## Configure `mod_shib` settings

Edit the file `/etc/httpd/conf.d/shib.conf` (installed as part of the `yum` package) and locate the `<Location>` tag (around line 49), which should look something like this:

```apache
<Location /secure>
```

Change the path of the directory to be secured to `/secured-by-saml`:

```apache
<Location /secured-by-saml>
```

## Restart HTTPD

Run the command

```console
casdev-samlsp# systemctl restart httpd
```

to restart the HTTPD server with the new configuration.

## Create example content

Edit the file `/var/www/html/index.php` and replace the call to `phpinfo()` with another link, like this:

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
        <p><big>Click <a href="secured-by-saml/index.php">here</a> for some
          content secured by username and password.</big></p>
    </div>
  </body>
</html>
```

Then create a directory, `/var/www/html/secured-by-saml`, and create the file `/var/www/html/secured-by-saml/index.php` with the following contents:

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
      <h1>Secured Content</h1>
      <p><big>This is some secure content. You should not be able to see it
       until you have entered your username and password.</big></p>
      <h2>Attributes Returned by SAML</h2>
      <?php
        echo "<pre>";

        if (array_key_exists('REMOTE_USER', $_SERVER)) {
            echo "REMOTE_USER = " . $_SERVER['REMOTE_USER'] . "<br>";
        }

        foreach ($_SERVER as $key => $value) {
            if (strpos($key, 'SAML_') === 0) {
                echo substr($key, 5) . " = " . $value . "<br>";
            }
        }

        echo "</pre>";
      ?>
    </div>
  </body>
</html>
```

The PHP code here will display environment variables that are used by `mod_shib` to pass attributes returned by the SAML IdP along to the web application.
