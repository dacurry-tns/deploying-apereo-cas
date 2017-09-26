---
title: Configure attribute resolution
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: building_server_ldap_resolution-release_configure-attribute-resolution.html
summary:
---

Now that the server has been configured to support attribute release, it must be configured to resolve (retrieve) the attributes to be released. Since the LDAP module has already been added to the server, all that is necessary to enable this is the definition of some additional properties.

## Configure Active Directory attribute resolution

Add the following lines to `etc/cas/config/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***) to enable CAS to resolve attributes from Active Directory:

```properties
cas.authn.attributeRepository.ldap[0].order:            0
cas.authn.attributeRepository.ldap[0].ldapUrl:          ldaps://zuul.newschool.edu
cas.authn.attributeRepository.ldap[0].userFilter:       sAMAccountName={user}
cas.authn.attributeRepository.ldap[0].baseDn:           ou=TNSUsers,dc=tns,dc=newschool,dc=edu
cas.authn.attributeRepository.ldap[0].bindDn:           cn=ldap_ssotest,ou=Service,ou=Users,ou=Enterprise Support,dc=tns,dc=newschool,dc=edu
cas.authn.attributeRepository.ldap[0].bindCredential:   xxxxxxxxxxxx
cas.authn.attributeRepository.ldap[0].attributes.cn:    uid
cas.authn.attributeRepository.ldap[0].attributes.displayName:   displayName
cas.authn.attributeRepository.ldap[0].attributes.givenName:     Formatted Name
cas.authn.attributeRepository.ldap[0].attributes.mail:  EmailAddress
cas.authn.attributeRepository.ldap[0].attributes.sn:    sn
cas.authn.attributeRepository.ldap[0].attributes.tnsGoogleAppsRole:     Role
cas.authn.attributeRepository.ldap[0].attributes.tnsIDNumber:   cn
```

The first six properties should be self-explanatory (or see the descriptions in the previous sections). Note that while we did not need to use a bind account to authenticate users against Active Directory, we do need to use one to resolve attributes.

The `.attributes.` properties specify, for each attribute, its name in the directory, and the name it should be given when sending it to the client application (the *mapped* name). For example, in the set of attributes above, the Active Directory attributes called `cn`, `displayName`, `givenName`, `mail`, `sn`, `tnsGoogleAppsRole`, and `tnsIDNumber` will be mapped to the names `uid`, `displayName`, `Formatted Name`, `EmailAddress`, `sn`, `Role`, and `cn` respectively when they are sent to client applications.

## Configure Luminis LDAP attribute resolution

Add the following lines to `etc/cas/config/cas.properties` to enable CAS to resolve attributes from Luminis LDAP:

```properties
cas.authn.attributeRepository.ldap[1].order:            1
cas.authn.attributeRepository.ldap[1].ldapUrl:          ldaps://janus.newschool.edu
cas.authn.attributeRepository.ldap[1].userFilter:       uid={user}
cas.authn.attributeRepository.ldap[1].baseDn:           ou=People,o=cp
cas.authn.attributeRepository.ldap[1].bindDn:           uid=ldap_ssotest,ou=People,o=cp
cas.authn.attributeRepository.ldap[1].bindCredential:   xxxxxxxxxxxx
cas.authn.attributeRepository.ldap[1].attributes.cn:    cn
cas.authn.attributeRepository.ldap[1].attributes.displayName:   displayName
cas.authn.attributeRepository.ldap[1].attributes.givenName:     Formatted Name
cas.authn.attributeRepository.ldap[1].attributes.mail:  EmailAddress
cas.authn.attributeRepository.ldap[1].attributes.sn:    sn
cas.authn.attributeRepository.ldap[1].attributes.udcid: UDC_IDENTIFIER
cas.authn.attributeRepository.ldap[1].attributes.uid:   uid
```

As above, the first six properties should be self-explanatory. The list of attributes to be released is similar to, but not the same as, the list for Active Directory, above.

One difference is that the two directories use different attributes for the same information. Luminis LDAP stores the username in the `uid` attribute and the student/employee ID number in the `cn` attribute. Active Directory on the other hand, stores the username in the `cn` attribute, and stores the student/employee ID number in a custom attribute called `tnsIDNumber`. To make things match up (the reason for this will become apparent below), the Active Directory configuration above switches things around to match Luminis LDAP by mapping `cn` to `uid` and `tnsIDNumber` as `cn`.

Another difference is that Active Directory has an attribute called `tnsGoogleAppsRole` (released as `Role`) that Luminis LDAP doesn't have, and Luminis LDAP has an attribute called `udcid` (released as `UDC_IDENTIFIER`) that Active Directory doesn't have.

## Configure an attribute merging strategy

Although CAS will only authenticate a user against the first directory (according to the evaluation order) in which the user is found, it will attempt to retrieve attributes from all configured repositories and then merge them together. The *merging strategy* determines what happens when CAS discovers the same attribute (based on the mapped name) in multiple repositories. The options are:

<table>
    <colgroup>
        <col width="25%" />
        <col width="75%" />
    </colgroup>
    <tbody>
        <tr>
            <td markdown="span">`REPLACE`</td>
            <td markdown="span">Overwrites the existing value (if any) with the new value. The attribute will contain the last value discovered.</td>
        </tr>
        <tr>
           <td markdown="span">`ADD`</td>
           <td markdown="span">Retain the existing value (if any), and ignore any subsequent values discovered for the same attribute. The attribute will contain the first value discovered.</td>
        </tr>
        <tr>
            <td markdown="span">`MERGE`</td>
            <td markdown="span">Combine all values into a single attribute, resulting in a comma-separated list of values.</td>
        </tr>
    </tbody>
</table>

In our case, we have a mix of users who are in Active Directory only, users who are in Luminis LDAP only, and users who are in both directories. Most of the time the duplicated attributes have the same value in both directories, but there are just enough exceptions to make `MERGE` a bad idea (applications that don't expect to receive multi-valued attributes don't handle them well). We have therefore (somewhat arbitrarily) decided that for users in both directories, the values of their attributes in Active Directory should "win," and since Active Directory is the first repository, we want to use the `ADD` strategy. So, add the following line to `etc/cas/config/cas.properties`:

```
cas.authn.attributeRepository.merger:                   ADD
```

Had we instead decided that Luminis LDAP should "win," `REPLACE` would be the correct strategy. Or, we could stick with the `ADD` strategy and change the evaluation order of the repositories.

## References

* [CAS 5: Configuration Properties: Authentication Attributes][casdoc-auth-attr-props]
* [CAS 5: Configuration Properties: Merging Strategies][casdoc-auth-merge-props]

{% include reflinks.md %}
