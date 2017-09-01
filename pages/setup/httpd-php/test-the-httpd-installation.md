---
title: Test the HTTPD installation
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_httpd-php_test-the-httpd-installation.html
summary:
---

{% include note.html content="The steps below are shown for ***casdev-casapp***; they should also be performed on ***casdev-samlsp*** with host names substituted as appropriate." %}

## Create a basic web page

Create the file `/var/www/html/index.php` with the following contents to make a basic web page that displays some simple content:

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
        <?php phpinfo(); ?>
    </div>
  </body>
</html>
```

{% include note.html content="Inclusion of the Bootstrap stylesheet is optional; it just makes the page a little more readable." %}

## Start HTTPD

Start the HTTPD server by running the command

```console
# systemctl start httpd
```

Review the contents of the log files in the `/var/log/httpd` directory for errors.

## Access the server

Open up a web browser and enter the HTTP URL of the server:

```
http://casdev-casapp.newschool.edu
```

Check that the server redirects the browser to the HTTPS version of the URL (the browser address bar should now display `https://casdev-casapp.newschool.edu`), and that you see something like this:

{% include image.html file="setup/httpd-php/fig04-test-example-webpage.png" alt="Browser Screen Shot" caption="Figure 4. The test example web page" %}

## Perform a TLS/SSL check on the servers (optional)

If ***casdev-casapp*** and ***casdev-samlsp*** are accessible from the Internet, use the [Qualys® SSL Labs SSL Server Test][qualys-ssltest] to check that TLS/SSL is correctly configured and that the servers receive an overall 'A' rating. If any other rating is received, check the test report for errors and correct them.

If ***casdev-casapp*** and ***casdev-samlsp*** are not accessible from the Internet, use the [`testssl.sh` command line tool ][testssl-sh] instead. This tool performs a similar battery of tests; the principal difference is that it doesn’t assign a letter grade to the overall results.

{% include reflinks.md %}
