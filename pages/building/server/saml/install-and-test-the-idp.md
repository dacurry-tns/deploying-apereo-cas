---
title: Install and test the IdP
last_updated: October 26, 2017
sidebar: main_sidebar
permalink: building_server_saml_install-and-test-the-idp.html
summary:
---

## Install and test on the master build server

Use the scripts [created earlier][building_server_install-and-test-the-cas-application] (or repeat the commands) to install the updated CAS application and configuration files on the master build server (***casdev-master***):

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# sh /opt/scripts/cassrv-install.sh
---Installing on casdev-master.newschool.edu
Installation complete.
casdev-master#  
```

Review the contents of the log files (`/var/log/tomcat/catalina.yyyy-mm-dd.out` and `/var/log/cas/cas.log`) for errors.

## Check that the IdP is working on the master build server

To check that the IdP functionality is working, use `curl` to request the IdP's metadata:

```console
casdev-master# curl -k https://casdev-master.newschool.edu:8443/cas/idp/metadata
<?xml version="1.0" encoding="UTF-8"?>
<EntityDescriptor xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:shibmd="urn:mace:shibboleth:metadata:1.0" xmlns:xml="http://www.w3.org/XML/1998/namespace" xmlns:mdui="urn:oasis:names:tc:SAML:metadata:ui" entityID="https://casdev.newschool.edu/idp">
    <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol urn:oasis:names:tc:SAML:1.1:protocol urn:mace:shibboleth:1.0">
        <Extensions>
            <shibmd:Scope regexp="false">newschool.edu</shibmd:Scope>
        </Extensions>
        <KeyDescriptor use="signing">
            <ds:KeyInfo>
                <ds:X509Data>
                    <ds:X509Certificate>MIIDMjCCAhqgAwIBAgIVAJ52X1Nr07ZSRcD/sf+/sru29lnuMA0GCSqGSIb3DQEB
CwUAMB8xHTAbBgNVBAMMFGNhc2Rldi5uZXdzY2hvb2wuZWR1MB4XDTE3MTAyMzE4
MzQxNloXDTM3MTAyMzE4MzQxNlowHzEdMBsGA1UEAwwUY2FzZGV2Lm5ld3NjaG9v
bC5lZHUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC9Ewa1ETMNN0rF
mpzNZln2x638hQjX78T5F78nk0/K37aHXaOUH7AgQshFuDVct+QfApur4FRtJl/H
F+LMJRSastrZDnjFEBMkb4iVsXZVQ+H+UsPEyVmPYfzTjtNwJFLsQfNStNmHF7AG
y+/3WxTa0KZKXq2yzGKIy9cnvmqzvve2PVmBOhwn3zFiT0ZgY+RoWzQVtIzkFst/
OATUg2c93LucM0x6Ec7VAYrH6sgBNjiP3NfsXv49mJhTJXuAkeWo3wGApR6R4ezm
meUm0HrNCZQ6dTRDGl5qZmax0ltCS2iJnwtd8BUSvwzZT1YrX57Qg2jrMZoo+SHR
FpCrQu79AgMBAAGjZTBjMB0GA1UdDgQWBBSDv5Ny5w80rMMx/VwYv5lsTVqYqzBC
BgNVHREEOzA5ghRjYXNkZXYubmV3c2Nob29sLmVkdYYhY2FzZGV2Lm5ld3NjaG9v
bC5lZHUvaWRwL21ldGFkYXRhMA0GCSqGSIb3DQEBCwUAA4IBAQAmhBGB9wOMJhZa
K9g1cX8bk9AcVeJAbltx/MUjiyopj98zy0DuuhUVPT1F2c36FFYzMx5JrCOfIwOl
BbDHSiSCTUapjQe2+McvTH2/Cvi2lNJ4rjlZpIdINJdzwEcTUnJYnUnWoO0oPALM
/V/IHlzqBR+lOtcv1wW19+WYT+BLzNMKc/ThmK2kZK88dpyLCyrxfiv1Om3dYNGM
oM6d+kQLuxM+h3qhnFVo/Xg3GJEImXoAYPS8A165y8kTJOa6FuvTR9kjmLoZ5E+9
36HVMrpjJZZO/XduOZ7k5qN20wRYMMyA7Lf85hgKItEmTJe/McrQgthbRaSsgY5q
nrndd+52</ds:X509Certificate>
                </ds:X509Data>
            </ds:KeyInfo>
        </KeyDescriptor>
        <KeyDescriptor use="encryption">
            <ds:KeyInfo>
                <ds:X509Data>
                    <ds:X509Certificate>MIIDMjCCAhqgAwIBAgIVAMX1YMhKcxKp/uraKDAmWA6u1AjnMA0GCSqGSIb3DQEB
CwUAMB8xHTAbBgNVBAMMFGNhc2Rldi5uZXdzY2hvb2wuZWR1MB4XDTE3MTAyMzE4
MzQxN1oXDTM3MTAyMzE4MzQxN1owHzEdMBsGA1UEAwwUY2FzZGV2Lm5ld3NjaG9v
bC5lZHUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCGo/8pOKu2ECEQ
XxHedB1RVTa4DGM5BYsubhrp6ANSS9eym2pGUi6hMJVPgdTaI3pnPecC8sk9DnQ5
O/7C7/utdit86bvDc4hUTJ1+W1OY2GrfS1QGqU2PFE0CtiJWtQp+g8ypBbXzcP6W
Zz3DRJMe1Jt44/fNoCgRHeiNVBWs3v2oNlpmA8A2+g9RKA9slrS4YuTkXNoTUYgo
JuZSDNhu0sZFx1WYElOrWs7ry7NCO1lJrl38UOGt9RFsF/7XRXSx4df2Hkr+XrIz
sVigUlxt1PiL8F03nH3tPMh4eIinmZ3F0r0vrFhrsc7Rzwkty4Bu/xEkA4mDE85k
YHy2UwjrAgMBAAGjZTBjMB0GA1UdDgQWBBSaNMHTF540kZ9XPGUuLb5EIM409TBC
BgNVHREEOzA5ghRjYXNkZXYubmV3c2Nob29sLmVkdYYhY2FzZGV2Lm5ld3NjaG9v
bC5lZHUvaWRwL21ldGFkYXRhMA0GCSqGSIb3DQEBCwUAA4IBAQAzHson1VU4/ZdC
pDXIUMACmN+WZPrgGQ+Xd3w179X1VrYmmU2KrWTu3Wq+svn0aUbEcWY+dDs7wFK2
gYXAYI8p4RY37n+Z/m8MyipZDHqZ6bloNsfjJ7j+L7yxLC5AQKgUpUON1kvsainI
zwr/W4Q0udFQzX1qxI6b9EF23V+8r4Hkl105Xh3bgkX5c52qUy71BoBDNvxvWXfT
q6NiXFEIX821Jc57xjqOWyD7tdfLnY5Imkb1ZQ5GlHHlWR6UKuZojIuBwT9Fotha
dgmL7gvBapJdfV1aNaz1MfC6REKnE5P+SZboEBGWpe8pGFSBKvyNAnfzCKNycSzH
/k3s3wIp</ds:X509Certificate>
                </ds:X509Data>
            </ds:KeyInfo>
        </KeyDescriptor>

        <!--
        <ArtifactResolutionService Binding="urn:oasis:names:tc:SAML:1.0:bindings:SOAP-binding"
                                   Location="https://casdev.newschool.edu/cas/idp/profile/SAML1/SOAP/ArtifactResolution" index="1"/>
        -->

        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://casdev.newschool.edu/cas/idp/profile/SAML2/POST/SLO"/>

        <NameIDFormat>urn:mace:shibboleth:1.0:nameIdentifier</NameIDFormat>
        <NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:transient</NameIDFormat>

        <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://casdev.newschool.edu/cas/idp/profile/SAML2/POST/SSO"/>
        <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST-SimpleSign" Location="https://casdev.newschool.edu/cas/idp/profile/SAML2/POST-SimpleSign/SSO"/>
        <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://casdev.newschool.edu/cas/idp/profile/SAML2/Redirect/SSO"/>
        <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://casdev.newschool.edu/cas/idp/profile/SAML2/SOAP/ECP"/>
    </IDPSSODescriptor>

    <!--
    <AttributeAuthorityDescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:1.1:protocol urn:oasis:names:tc:SAML:2.0:protocol">
        <Extensions>
            <shibmd:Scope regexp="false">newschool.edu</shibmd:Scope>
        </Extensions>
        <KeyDescriptor use="signing">
            <ds:KeyInfo>
                <ds:X509Data>
                    <ds:X509Certificate>MIIDMjCCAhqgAwIBAgIVAJ52X1Nr07ZSRcD/sf+/sru29lnuMA0GCSqGSIb3DQEB
CwUAMB8xHTAbBgNVBAMMFGNhc2Rldi5uZXdzY2hvb2wuZWR1MB4XDTE3MTAyMzE4
MzQxNloXDTM3MTAyMzE4MzQxNlowHzEdMBsGA1UEAwwUY2FzZGV2Lm5ld3NjaG9v
bC5lZHUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC9Ewa1ETMNN0rF
mpzNZln2x638hQjX78T5F78nk0/K37aHXaOUH7AgQshFuDVct+QfApur4FRtJl/H
F+LMJRSastrZDnjFEBMkb4iVsXZVQ+H+UsPEyVmPYfzTjtNwJFLsQfNStNmHF7AG
y+/3WxTa0KZKXq2yzGKIy9cnvmqzvve2PVmBOhwn3zFiT0ZgY+RoWzQVtIzkFst/
OATUg2c93LucM0x6Ec7VAYrH6sgBNjiP3NfsXv49mJhTJXuAkeWo3wGApR6R4ezm
meUm0HrNCZQ6dTRDGl5qZmax0ltCS2iJnwtd8BUSvwzZT1YrX57Qg2jrMZoo+SHR
FpCrQu79AgMBAAGjZTBjMB0GA1UdDgQWBBSDv5Ny5w80rMMx/VwYv5lsTVqYqzBC
BgNVHREEOzA5ghRjYXNkZXYubmV3c2Nob29sLmVkdYYhY2FzZGV2Lm5ld3NjaG9v
bC5lZHUvaWRwL21ldGFkYXRhMA0GCSqGSIb3DQEBCwUAA4IBAQAmhBGB9wOMJhZa
K9g1cX8bk9AcVeJAbltx/MUjiyopj98zy0DuuhUVPT1F2c36FFYzMx5JrCOfIwOl
BbDHSiSCTUapjQe2+McvTH2/Cvi2lNJ4rjlZpIdINJdzwEcTUnJYnUnWoO0oPALM
/V/IHlzqBR+lOtcv1wW19+WYT+BLzNMKc/ThmK2kZK88dpyLCyrxfiv1Om3dYNGM
oM6d+kQLuxM+h3qhnFVo/Xg3GJEImXoAYPS8A165y8kTJOa6FuvTR9kjmLoZ5E+9
36HVMrpjJZZO/XduOZ7k5qN20wRYMMyA7Lf85hgKItEmTJe/McrQgthbRaSsgY5q
nrndd+52</ds:X509Certificate>
                </ds:X509Data>
            </ds:KeyInfo>
        </KeyDescriptor>
        <AttributeService Binding="urn:oasis:names:tc:SAML:1.0:bindings:SOAP-binding" Location="https://casdev.newschool.edu/cas/idp/profile/SAML1/SOAP/AttributeQuery"/>
        <AttributeService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://casdev.newschool.edu/cas/idp/profile/SAML2/SOAP/AttributeQuery"/>
    </AttributeAuthorityDescriptor>
    -->

    <!--
    <Organization>
        <OrganizationName xml:lang="en">Institution Name</OrganizationName>
        <OrganizationDisplayName xml:lang="en">Institution DisplayName</OrganizationDisplayName>
        <OrganizationURL xml:lang="en">URL</OrganizationURL>
    </Organization>
    <ContactPerson contactType="administrative">
        <GivenName>John Smith</GivenName>
        <EmailAddress>jsmith@example.org</EmailAddress>
    </ContactPerson>
    <ContactPerson contactType="technical">
        <GivenName>John Smith</GivenName>
        <EmailAddress>jsmith@example.org</EmailAddress>
    </ContactPerson>
    <ContactPerson contactType="support">
        <GivenName>IT Services Support</GivenName>
        <EmailAddress>support@example.org</EmailAddress>
    </ContactPerson>
    -->
</EntityDescriptor>
casdev-master#  
```

You should receive a relatively lengthy XML document back.

{% include note.html content="The `-k` option to `curl` tells it not to verify the SSL certificate of the server; this is necessary because the server is identifying itself as ***casdev.newschool.edu***, not ***casdev-master.newschool.edu***." %}

## Copy CAS-generated IdP metadata to the overlay template

When the CAS server was started with IdP support for the first time (above), it generated IdP-specific signing and encryption keys and certificates to be used when communicating with SAML2 clients. The server wrote copies of these keys and certificates, as well as the IdP metadata, to files in `/etc/cas/saml`:

```console
casdev-master# ls -asl /etc/cas/saml
total 28
4 drwxrwx---. 2 root   tomcat 4096 Mmm dd hh:mm .
0 drwxr-x---. 5 root   tomcat   45 Mmm dd hh:mm ..
4 -rw-rw----. 1 tomcat tomcat 1168 Mmm dd hh:mm idp-encryption.crt
4 -rw-rw----. 1 tomcat tomcat 1675 Mmm dd hh:mm idp-encryption.key
8 -rw-rw----. 1 tomcat tomcat 7383 Mmm dd hh:mm idp-metadata.xml
4 -rw-rw----. 1 tomcat tomcat 1168 Mmm dd hh:mm idp-signing.crt
4 -rw-rw----. 1 tomcat tomcat 1679 Mmm dd hh:mm idp-signing.key
casdev-master#  
```

To make sure that these files are the same across all the CAS servers (so that any back-end server can talk to any client), copy them into the overlay template's `etc/cas` directory:

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# cp /etc/cas/saml/idp-* etc/cas/saml
```

## Install on the CAS servers

Once everything seems to be running correctly on the master build server (further testing will have to wait until we [build the SAML client][building_samlclient_overview]), copy the updated Tomcat settings to the CAS servers:

```console
casdev-master# for host in srv01 srv02 srv03
> do
> scp -p /etc/tomcat/server.xml casdev-${host}:/etc/tomcat/server.xml
> done
casdev-master#  
```

Then copy the new CAS application files to the CAS servers using the scripts [created earlier][building_server_install-and-test-the-cas-application]:

```console
casdev-master# sh /opt/scripts/cassrv-tarball.sh
casdev-master# for host in srv01 srv02 srv03
> do
> scp -p /tmp/cassrv-files.tgz casdev-${host}:/tmp/cassrv-files.tgz
> scp -p /opt/scripts/cassrv-install.sh casdev-${host}:/tmp/cassrv-install.sh
> ssh casdev-${host} sh /tmp/cassrv-install.sh
> done
casdev-master#  
```

## Check that the IdP is working on the CAS servers

Use the `curl` command again to check that the IdP is working on the CAS servers. This time the `-k` option is not necessary:

```console
casdev-master# curl https://casdev.newschool.edu/cas/idp/metadata
(XML output)
casdev-master#  
```

{% include reflinks.md %}
{% include links.html %}
