---
title: Configure Duo authentication
last_updated: December 20, 2017
sidebar: main_sidebar
permalink: building_server_mfa_configure-duo-authentication.html
summary:
---

Configuring Duo authentication requires setting up a new application to be protected via the Duo administrator console, and then copying some of the information from that configuration to the `cas.properties` file.

## Create a new Duo protected application

To create a new Duo protected application:

1. Log into the Duo administrator console.
2. Select "Applications > Protect an Application" from the links on the left side of the window.
3. Find the entry for **CAS (Central Authentication Service)** in the list of applications and click on "Protect this Application".
4. Scroll down to the **Settings** section and set the application name to something meaningful, for example, `CASDev CAS Server`.
5. Make any other settings changes as appropriate.
6. Click "Save Changes".

Don't log out of the administrator console yet; some of the information there must be copied to `cas.properties`.

## Configure Duo authentication properties

Add the following settings to `etc/cas/config/cas.properties` in the `cas-overlay-template` directory on the master build server (***casdev-master***) to enable Duo authentication:

```properties
cas.authn.mfa.duo[0].duoApiHost:        api-a1b2c3d4.duosecurity.com
cas.authn.mfa.duo[0].duoIntegrationKey: DIYQCAFU5Q5UCD24J00R
cas.authn.mfa.duo[0].duoSecretKey:      FeTtpcOFyDxvtrtOXqma74DztXf7INRDKENMhAOF
cas.authn.mfa.duo[0].duoApplicationKey: 3d787231f9b9e128a9b94647b6e968f1fe0deddc
```

The `[0]` in the property names indicates that this is the first Duo provider to be configured. Additional providers (for example, if there are different providers for different locations or different groups of users) will be `[1]`, `[2]`, etc.

The `duoApiHost`, `duoIntegrationKey`, and `duoSecretKey` values should be copied from the **Details** section of the protected application page in the Duo administrator console (see above).

The `duoApplicationKey` value is a string, at least 40 characters long, that is generated locally and is *not* shared with Duo. The CAS documentation offers the procedure below for generating this string:

<div class="language-console highlighter-rouge"><pre class="highlight"><code><span class="ni">casdev-master# </span><span class="nc">python</span><span class="kv">
</span>Python 2.7.5 (default, Aug  2 2016, 04:20:16)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-4)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
<span class="ni">&gt;&gt;&gt; </span><span class="nc">import</span><span class="kv"> os, hashlib</span>
<span class="ni">&gt;&gt;&gt; </span><span class="nc">print</span><span class="kv"> hashlib.sha1(os.urandom(32)).hexdigest()</span>
3d787231f9b9e128a9b94647b6e968f1fe0deddc
<span class="ni">&gt;&gt;&gt; </span><span class="nc">exit()</span>
<span class="ni">casdev-master# </span><span class="kv">
</span></code></pre>
</div>

## References

* [CAS 5: Configuration Properties: Duo Security][casdoc-mfa-duo-props]

{% include reflinks.md %}
