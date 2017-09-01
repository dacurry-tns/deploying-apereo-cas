---
title: Adding LDAP support
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_ldap_overview.html
summary: Multiple LDAP directories will be used to authenticate users and to collect user attributes (ID numbers, names, group memberships, etc.) and make them available to client applications.
---

Although the default user credentials (`casuser` / `Mellon`) provided by the CAS server are useful for testing, we really want users to enter their own individual usernames and passwords. For the CAS server to support that in our environment, it has to be able to authenticate users against one or more LDAP directories. Many services that we use also require other information about users besides their username and password, such as their first and last name, student or employee ID numbers, group memberships, and so on. We store this information in LDAP as well, so the CAS server has to know how to retrieve it and send it to the client service.

In this section, we will add LDAP support to the CAS server to enable it to do three things:

1. **Authentication.** Prompt the user for his or her username and password, and validate that the provided password is indeed correct. At this stage, the user account is also checked to ensure that it is not suspended or disabled. At the conclusion of the authentication process, the CAS server will have identified a *security principal*. A CAS principal contains a unique identifier by which the authenticated user will be known to all requesting services. A principal also contains optional attributes that may be released to services to support authorization and personalization.  
2. **Attribute resolution.** Specific attributes about the principal are collected from one or more sources and combined into a single set of attributes using any of several different combining strategies (merging, replacing, adding, etc.).
3. **Attribute release.** The process of defining how attributes are selected and provided to a given application in the final CAS response.

## References

* [CAS 5: Configuring Authentication Components][casdoc-config-auth]
* [CAS 5: Configuring Principal Resolution][casdoc-config-prin-res]
* [CAS 5: Attribute Resolution][casdoc-attrib-res]
* [CAS 5: Attribute Release][casdoc-attrib-rel]

{% include reflinks.md %}
{% include links.html %}
