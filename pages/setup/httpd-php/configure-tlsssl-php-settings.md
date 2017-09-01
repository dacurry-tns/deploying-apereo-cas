---
title: Configure TLS/SSL settings
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_httpd-php_configure-tlsssl-php-settings.html
summary:
---

Apache HTTPD’s TLS/SSL settings must be configured and a TLS/SSL certificate must be provided to ensure that all communications with the CAS server occur over a secure channel.

{% include note.html content="The steps below are shown for ***casdev-casapp***; they should also be performed on ***casdev-samlsp*** with host names substituted as appropriate." %}

##	Generate private keys and certificate signing requests

The private key and certificate signing request are generated using the `openssl` command. Run the commands

```console
casdev-casapp# cd /etc/pki/tls/private
casdev-casapp# openssl req -nodes -newkey rsa:2048 -sha256 -keyout casapp.key -out casapp.csr
Generating a 2048 bit RSA private key
.........+++
.............+++
writing new private key to 'casapp.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []: New York
Locality Name (eg, city) [Default City]: New York
Organization Name (eg, company) [Default Company Ltd]: The New School
Organizational Unit Name (eg, section) []: IT
Common Name (eg, your name or your server's hostname) []: casdev-casapp.newschool.edu
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
casdev-casapp#  
```

to generate a private key and certificate signing request. (Replace the contents of the Distinguished Name fields with values appropriate for your organization.) Submit the certificate signing request (`casapp.csr`) to your certificate authority to obtain a certificate.

When the certificate comes back from the certificate authority, copy it and any intermediate certificate(s) into `/etc/pki/tls/certs`, saving them as `casapp.crt`, `casapp-intermediate.crt`, etc. If your certificate authority offers multiple certificate formats, opt for the PEM format, which looks like:

```
-----BEGIN CERTIFICATE-----
AQEFAAOCAQ8AMIIBCgKCAQEAtGCKiysqhQF4/AA5Pvi7EIIRqbtVx/IF0CAFK8lv
6uDJDHjd7bSNhhzYJxUNCdN0DacYT5wI/s4n3mLEXQrIt0KsUdPD+s7qP9Lw05hI
WaG7KhP6RZ+UtWSvHwIZJUHvlJvh2GlARw/XwV3iHG3mxfl5nCLNihAR9S1r2qEY
...several more lines of base64-encoded data...
-----END CERTIFICATE----
```

## Configure HTTPD settings

Edit the file `/etc/httpd/conf/httpd.conf` and locate the `ServerName` directive (around line 95), which should look something like this:

```apache
#ServerName www.example.com:80
```

Uncomment the line and replace `www.example.com:80` with `casdev-casapp.newschool.edu`, and then add a `UseCanonicalName` directive on the next line, like this:

```apache
ServerName casdev-casapp.newschool.edu
UseCanonicalName on
```

Then, at the bottom of the file, add the following lines:

```apache
<VirtualHost *:80>
    Redirect permanent / https://casdev-casapp.newschool.edu/
</VirtualHost>
```

to automatically redirect any HTTP connections (port 80) to HTTPS connections (port 443), which will help ensure that all communications occur over a secure communications channel.

## Configure TLS/SSL settings

Edit the file `/etc/httpd/conf.d/ssl.conf` and locate the `SSLProtocol` directive (around line 75) and the `SSLCipherSuite` directive (around line 80), the two of which should look something like this:

```apache
SSLProtocol all -SSLv2
SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5:!SEED:!IDEA
```

Change the values of these two directives, and add an `SSLHonorCipherOrder` directive, so that it all looks like this:

```apache
SSLProtocol all -SSLv2 -SSLv3
SSLCipherSuite ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS
SSLHonorCipherOrder on
```

To obtain the most up-to-date values for these three attributes, use the [Mozilla SSL Configuration Generator][mozilla-ssl-gen] and select "Apache" and "Intermediate." Then copy and paste the values given by the generator into the configuration above.

Then locate the `SSLCertificateFile`, `SSLCertificateKeyFile`, and `SSLCertificateChainFile` directives (around lines 100-116) and set them to the names of the server certificate file, private key file, and (if applicable) intermediate certificate file:

```apache
SSLCertificateFile /etc/pki/tls/certs/casapp.crt
SSLCertificateKeyFile /etc/pki/tls/private/casapp.key
SSLCertificateChainFile /etc/pki/tls/certs/casapp-intermediate.crt
```
## Check the HTTPD Configuration

Check that the HTTPD configuration files edited above do not have any syntax errors by running the command

```console
casdev-casapp# apachectl configtest
Syntax OK
casdev-casapp#
```

## Configure PHP settings

Edit the file `/etc/php.ini` and locate the `date.timezone` setting (around line 878), which should look something like:

```php
;date.timezone =
```

Uncomment the line and set the value to the local time zone:

```php
date.timezone = America/New_York
```

The list of supported time zone names is available from the [PHP List of Supported Timezones][php-timezones].

{% include reflinks.md %}
