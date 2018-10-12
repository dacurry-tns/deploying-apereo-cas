---
title: Configure Google single sign-on
last_updated: October 12, 2018
sidebar: main_sidebar
permalink: googleapps_configure-google-sso.html
summary:
---

Once the CAS side of things has been set up, the Google side has to be configured.

## Configure SSO URLs

To configure a Google Apps domain to use the CAS server for user authentication:

1. Log in to the Google Admin Console for the domain to be configured.
2. Go to **Security > Set up single sign-on (SSO)**.
3. Check the **Setup SSO with third party identity provider** box
4. Enter the CAS login URL (`https://casdev.newschool.edu/cas/login`) in the **Sign-in page URL** blank. This should be the URL of the CAS login endpoing, ***not*** the URL of the CAS SAML2 IdP endpoint.
5. Enter the CAS logout URL (`https://casdev.newschool.edu/cas/logout`) in the **Sign-out page URL** blank.
6. Enter the URL a user should be directed to when changing his or her password in the **Change password URL** blank. This may or may not be a CAS endpoint, depending on whether the CAS password management feature has been configured.

{% include important.html content="All URLs must be entered, and they must all use HTTPS." %}

## Upload verification certificate

The X.509 file [created earlier][googleapps_generate-keys-and-certificates] has to be uploaded so that Google can verify sign-in requests.

After configuring the URLs and uploading the certificate, click the **SAVE** button.

## References

* [Google: Service provider SSO set up][google-sp-sso-setup]
* [Google: SAML key and verification certificate][google-saml-key-and-cert]

{% include reflinks.md %}
{% include links.html %}
