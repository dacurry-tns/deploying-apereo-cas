---
title: Generate keys and certificates
last_updated: October 12, 2018
sidebar: main_sidebar
permalink: googleapps_generate-keys-and-certificates.html
summary:
---

Google's SAML2 implementation requires that the SAML assertions exchanged with the CAS server be encrypted and signed. Therefore, it's necessary to generate a public/private key pair and an X.509 certificate for this purpose.

## Use OpenSSL to generate the keys and certificate

Although it's not the only method, OpenSSL is perhaps the easiest way to create the keys and certificate. Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# mkdir etc/cas/google
casdev-master# cd etc/cas/google
casdev-master# openssl genrsa -out privatekey.pem 2048
Generating RSA private key, 2048 bit long modulus
.....................................................................+++
........................................................................................+++
e is 65537 (0x10001)
casdev-master# openssl rsa -in privatekey.pem -pubout -outform DER -out publickey.der
writing RSA key
casdev-master# openssl pkcs8 -topk8 -inform PEM -outform DER -in privatekey.pem -out privatekey.der -nocrypt
casdev-master# openssl req -new -x509 -key privatekey.pem -out x509.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []:New York
Locality Name (eg, city) [Default City]:New York
Organization Name (eg, company) [Default Company Ltd]:The New School
Organizational Unit Name (eg, section) []:IT
Common Name (eg, your name or your server's hostname) []:newschool.edu
Email Address []:

```

{% include note.html content="The CAS documentation provides slightly outdated instructions for creating the keys. To ensure compatibility, follow the directions from Google's documentation (linked below). The instructions above are based on the Google instructions." %}

When providing the Distinguished Name information for the X.509 certificate, use the same information you use when creating an SSL certificate. For the Common Name field, use the domain name of the Google Apps instance.

Although it's possible to include the keys and certificate in the CAS WAR file (or somewhere else on the classpath), we will keep them in the `etc/cas/google` directory in the overlay, which will result in their getting copied to `/etc/cas/google` when the server is deployed. This helps to ensure that they are not accidentally deleted.

## Configure Google Apps properties

Add the following settings to `etc/cas/config/cas.properties` in the `cas-overlay-template` directory:

```properties
cas.googleApps.privateKeyLocation:  file:/etc/cas/google/privatekey.der
cas.googleApps.publicKeyLocation:   file:/etc/cas/google/publickey.der
cas.googleApps.keyAlgorithm:        RSA
```


## References

* [CAS 5: Google Apps Integration][casdoc-googleapps]
* [CAS 5: Configuration Properties: Google Apps][casdoc-config-googleapps]
* [Google: Generate Keys and Certificates for SSO][google-gen-keys-certs]

{% include reflinks.md %}
{% include links.html %}
