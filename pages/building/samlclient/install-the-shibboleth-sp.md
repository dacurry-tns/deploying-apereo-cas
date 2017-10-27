---
title: Install the Shibboleth SP
last_updated: October 27, 2017
sidebar: main_sidebar
permalink: building_samlclient_install-the-shibboleth-sp.html
summary:
---

The Shibboleth Service Provider (SP) allows an Apache web server to interact with a SAML Identity Provider (IdP) via the SAML2 protocol. The SP is comprised of an Apache HTTPD module (`mod_shib`) and a system daemon (`shibd`) that handles state management and most of the actual SAML processing (the module communicates with the daemon; the daemon communicates with the IdP). Red Hat does not offer this software, but the Shibboleth Consortium uses the [OpenSUSE Build Service][opensuse-build-svc] to distribute its packages, so we can still install it with `yum`.

{% include note.html content="The steps in this section should be performed on the client server (***casdev-samlsp***), not the master build server (***casdev-master***)." %}

## Add the Shibboleth repository to `yum`

Before we can use `yum` to install the Shibboleth SP, we have to teach it about the Shibboleth repository from the OpenSUSE Build Service. Run the command

```console
casdev-samlsp# curl http://download.opensuse.org/repositories/security:/shibboleth/CentOS_7/security:shibboleth.repo -o /etc/yum.repos.d/security\:shibboleth.repo
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   266  100   266    0     0   1225      0 --:--:-- --:--:-- --:--:--  1231
casdev-samlsp#  
```

to download the repo configuration file.

{% include important.html content="From the [Shibboleth Wiki](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPLinuxRPMInstall): \"A special note applies to Red Hat 7 and probably all future versions: because of Red Hat's licensing restrictions, it's now impossible for the build service to target Red Hat 7 directly. However, CentOS is an identical system, and the packages for it work on the equivalent Red Hat versions, so Red Hat 7 deployments should rely on the CentOS 7 package repository.\"" %}

## Install the Shibboleth SP

Install the Shibboleth SP and its dependencies by running the commands

```console
casdev-samlsp# yum -y install shibboleth
```

## Create a TLS/SSL certificate for the SP

The SP uses its own TLS/SSL certificate for signing and encrypting communications with the IdP. Run the commands

```console
casdev-samlsp# cd /etc/shibboleth
casdev-samlsp# ./keygen.sh -h casdev-samlsp.newschool.edu -e https://casdev.newschool.edu/shibboleth -f -u shibd -y 10
Generating a 3072 bit RSA private key
.........++
...................................................++
writing new private key to './sp-key.pem'
-----
casdev-samlsp#  
```

Any error messages about being unable to remove `./sp-key.pem` or `./sp-cert.pem` may be safely ignored.

## Configure `systemd` to start `shibd`

RHEL 7 uses `systemd` to manage system resources. Run the command

```console
casdev-samlsp# systemctl enable shibd
```

to enable the `shibd` service in `systemd`. This will cause `systemd` to start `shibd` at system boot time. Additionally, the following commands may now be used to manually start, stop, restart, and check the status of the `shibd` service:

```console
# systemctl start shibd
# systemctl stop shibd
# systemctl restart shibd
# systemctl status shibd
```

## Test that HTTPD and `shibd` can communicate

Before configuring the SP further, check that Apache HTTP and `shibd` are able to communicate. Run the commands

```console
casdev-samlsp# systemctl start shibd
casdev-samlsp# systemctl restart HTTPD
```

to start the `shibd` daemon and restart HTTPD to load the `mod_shib` module. Then use `curl` to retrieve the SP's status page:

```console
casdev-samlsp# curl -k https://127.0.0.1/Shibboleth.sso/Status
<StatusHandler time='2017-10-25T19:12:42Z'><Version Xerces-C='3.1.1' XML-Tooling-C='1.6.0' XML-Security-C='1.7.3' OpenSAML-C='2.6.0' Shibboleth='2.6.0'/><NonWindows sysname='Linux' nodename='casdev-samlsp.newschool.edu' release='3.10.0-693.2.2.el7.x86_64' version='#1 SMP Sat Sep 9 03:55:24 EDT 2017' machine='x86_64'/><SessionCache><OK/></SessionCache><Application id='default' entityID='https://sp.example.org/shibboleth'/><Handlers><Handler type='ArtifactResolutionService' Location='/Artifact/SOAP' Binding='urn:oasis:names:tc:SAML:2.0:bindings:SOAP'/><Handler type='AssertionConsumerService' Location='/SAML2/POST' Binding='urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST'/><Handler type='AssertionConsumerService' Location='/SAML2/POST-SimpleSign' Binding='urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST-SimpleSign'/><Handler type='AssertionConsumerService' Location='/SAML2/Artifact' Binding='urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Artifact'/><Handler type='AssertionConsumerService' Location='/SAML2/ECP' Binding='urn:oasis:names:tc:SAML:2.0:bindings:PAOS'/><Handler type='AssertionConsumerService' Location='/SAML/POST' Binding='urn:oasis:names:tc:SAML:1.0:profiles:browser-post'/><Handler type='AssertionConsumerService' Location='/SAML/Artifact' Binding='urn:oasis:names:tc:SAML:1.0:profiles:artifact-01'/><Handler type='SessionInitiator' Location='/Login'/><Handler type='SingleLogoutService' Location='/SLO/SOAP' Binding='urn:oasis:names:tc:SAML:2.0:bindings:SOAP'/><Handler type='SingleLogoutService' Location='/SLO/Redirect' Binding='urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect'/><Handler type='SingleLogoutService' Location='/SLO/POST' Binding='urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST'/><Handler type='SingleLogoutService' Location='/SLO/Artifact' Binding='urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Artifact'/><Handler type='LogoutInitiator' Location='/Logout'/><Handler type='MetadataGenerator' Location='/Metadata'/><Handler type='Status' Location='/Status'/><Handler type='Session' Location='/Session'/><Handler type='DiscoveryFeed' Location='/DiscoFeed'/></Handlers><md:KeyDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" use="signing"><ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#"><ds:KeyName>casdev-samlsp.newschool.edu</ds:KeyName><ds:KeyName>https://casdev.newschool.edu/shibboleth</ds:KeyName><ds:X509Data><ds:X509SubjectName>CN=casdev-samlsp.newschool.edu</ds:X509SubjectName><ds:X509Certificate>MIIEQTCCAqmgAwIBAgIJAJgyfve+2p4MMA0GCSqGSIb3DQEBCwUAMCYxJDAiBgNV
BAMTG2Nhc2Rldi1zYW1sc3AubmV3c2Nob29sLmVkdTAeFw0xNzEwMjUxOTA5NTNa
Fw0yNzEwMjMxOTA5NTNaMCYxJDAiBgNVBAMTG2Nhc2Rldi1zYW1sc3AubmV3c2No
b29sLmVkdTCCAaIwDQYJKoZIhvcNAQEBBQADggGPADCCAYoCggGBAKnAVWGK7S1f
nmL41kvXlU9l2BDQNKEpoEnu428Bg8Az/5e1wxs2eMk9XymPobEJ5LlT0Fyr4nSl
NqRSHkCOFdoll+W8qfqn3AxgYaGDccU+JREp2uz8FYDpAwFsC/3bGWVrKW6aUW6T
sAoQwdcL2XlokQ3NkdwimYv6gB2sP8kNje7G8gl1kEodu6QucDwChvHHTF5MQuPz
5L2iWBif2ZFBE7+AokIYL3rZCUSVviq6e77hJ2p0l4Rtx2I20AaHKaOFGITwtELx
mIVzJ9747lEq6xV9VeFFG7wmJqFioy39hPoKL9rtMhkGg78LsX6famSjYqUW12qs
QjyazOUd0ve3C8WBX/PVIA3I3WpQTpgT/xIZSbuHUa7fYE5+BRvcZOaRVk6WfHja
ABCx43D5f6FPHj3pfcsGZzKrT26QJYeIIcNgsgH+KQAszjfIYF1iJNCkJQGGkbQX
DL+Fvr7PaLOWjf31A85KSHWXe9F7cFovwk+l6Q8RuT4al8sL9ibYcwIDAQABo3Iw
cDBPBgNVHREESDBGghtjYXNkZXYtc2FtbHNwLm5ld3NjaG9vbC5lZHWGJ2h0dHBz
Oi8vY2FzZGV2Lm5ld3NjaG9vbC5lZHUvc2hpYmJvbGV0aDAdBgNVHQ4EFgQUlo8B
+EW+irsywPjzuc8cBeLvXU8wDQYJKoZIhvcNAQELBQADggGBAKh7J3VZAYNs5Lr3
ox7rvz9Vhx5ZzvZ28j6TlTFvtbh4OSNYlP3G3o627MtORY/bPzm63bQrfq53OTOf
JiqMENEOGXrqBbFqRfR32P75BWfXsh2ZSxdGXU6O9czFpjrycAKZgWv9U2OYFpyb
m14RzSqQq34pyL8nScJ2dO5cK/Ei5SeL76U8Jf68fVVhL6eEZ3eV93jtUafLW4r6
ouiO9v26d+W6hnzk5R0ntitNgkCudcpFH/heDXawbxhXOkEHbAQ1ggqdE/vnWVfx
PKxvhKw6UW1PsaYoy+yiXwzX3rIxLGTfnqJ8cdb4gjDE8tXOxMjVdbWeFTY5aDWG
Nv9KkJ9W5dEz1s91enceM9XzINvca2d/qa7hflEEHnWNhXKIQ8BEXKjEPFFkcclo
r5hpkVNVVGEC2ZA1nSQlrDBMpf/3uDgPxVh474vuDTJHvcEXpfNqEITBdxO2t7dV
HGiljHQvACTaqHAL0sYUpZPVk3CE2RV+B0m3jugIlnT3VB7tFQ==
</ds:X509Certificate></ds:X509Data></ds:KeyInfo></md:KeyDescriptor><md:KeyDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" use="encryption"><ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#"><ds:KeyName>casdev-samlsp.newschool.edu</ds:KeyName><ds:KeyName>https://casdev.newschool.edu/shibboleth</ds:KeyName><ds:X509Data><ds:X509SubjectName>CN=casdev-samlsp.newschool.edu</ds:X509SubjectName><ds:X509Certificate>MIIEQTCCAqmgAwIBAgIJAJgyfve+2p4MMA0GCSqGSIb3DQEBCwUAMCYxJDAiBgNV
BAMTG2Nhc2Rldi1zYW1sc3AubmV3c2Nob29sLmVkdTAeFw0xNzEwMjUxOTA5NTNa
Fw0yNzEwMjMxOTA5NTNaMCYxJDAiBgNVBAMTG2Nhc2Rldi1zYW1sc3AubmV3c2No
b29sLmVkdTCCAaIwDQYJKoZIhvcNAQEBBQADggGPADCCAYoCggGBAKnAVWGK7S1f
nmL41kvXlU9l2BDQNKEpoEnu428Bg8Az/5e1wxs2eMk9XymPobEJ5LlT0Fyr4nSl
NqRSHkCOFdoll+W8qfqn3AxgYaGDccU+JREp2uz8FYDpAwFsC/3bGWVrKW6aUW6T
sAoQwdcL2XlokQ3NkdwimYv6gB2sP8kNje7G8gl1kEodu6QucDwChvHHTF5MQuPz
5L2iWBif2ZFBE7+AokIYL3rZCUSVviq6e77hJ2p0l4Rtx2I20AaHKaOFGITwtELx
mIVzJ9747lEq6xV9VeFFG7wmJqFioy39hPoKL9rtMhkGg78LsX6famSjYqUW12qs
QjyazOUd0ve3C8WBX/PVIA3I3WpQTpgT/xIZSbuHUa7fYE5+BRvcZOaRVk6WfHja
ABCx43D5f6FPHj3pfcsGZzKrT26QJYeIIcNgsgH+KQAszjfIYF1iJNCkJQGGkbQX
DL+Fvr7PaLOWjf31A85KSHWXe9F7cFovwk+l6Q8RuT4al8sL9ibYcwIDAQABo3Iw
cDBPBgNVHREESDBGghtjYXNkZXYtc2FtbHNwLm5ld3NjaG9vbC5lZHWGJ2h0dHBz
Oi8vY2FzZGV2Lm5ld3NjaG9vbC5lZHUvc2hpYmJvbGV0aDAdBgNVHQ4EFgQUlo8B
+EW+irsywPjzuc8cBeLvXU8wDQYJKoZIhvcNAQELBQADggGBAKh7J3VZAYNs5Lr3
ox7rvz9Vhx5ZzvZ28j6TlTFvtbh4OSNYlP3G3o627MtORY/bPzm63bQrfq53OTOf
JiqMENEOGXrqBbFqRfR32P75BWfXsh2ZSxdGXU6O9czFpjrycAKZgWv9U2OYFpyb
m14RzSqQq34pyL8nScJ2dO5cK/Ei5SeL76U8Jf68fVVhL6eEZ3eV93jtUafLW4r6
ouiO9v26d+W6hnzk5R0ntitNgkCudcpFH/heDXawbxhXOkEHbAQ1ggqdE/vnWVfx
PKxvhKw6UW1PsaYoy+yiXwzX3rIxLGTfnqJ8cdb4gjDE8tXOxMjVdbWeFTY5aDWG
Nv9KkJ9W5dEz1s91enceM9XzINvca2d/qa7hflEEHnWNhXKIQ8BEXKjEPFFkcclo
r5hpkVNVVGEC2ZA1nSQlrDBMpf/3uDgPxVh474vuDTJHvcEXpfNqEITBdxO2t7dV
HGiljHQvACTaqHAL0sYUpZPVk3CE2RV+B0m3jugIlnT3VB7tFQ==
</ds:X509Certificate></ds:X509Data></ds:KeyInfo></md:KeyDescriptor><Status><OK/></Status></StatusHandler>
casdev-samlsp#  
```

The details of the XML document returned by this command aren't terribly important since the SP hasn't been configured yet.

{% include note.html content="The status page is protected by an IP address ACL, so the `curl` command must be run from the server against the `localhost` IP address (`127.0.0.1`). The `-k` option to `curl` is also needed, since the server's SSL certificate is advertising a different host name." %}

## References

* [Shibboleth SP: NativeSPLinuxRPMInstall][shibboleth-splinuxrpm]
* [ShibInstallFest: Linux Service Provider (RHEL 7.0)][shibinstallfest]

{% include reflinks.md %}
{% include links.html %}
