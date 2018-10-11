---
title: Introduction
last_updated: September 11, 2018
sidebar: main_sidebar
permalink: introduction_overview.html
summary: This document provides step-by-step instructions for setting up an Apereo CAS 5 environment. It was created during the process of building a brand new development environment to experiment with many of the new features in this release.
---

{% include callout.html type="primary" content="**CAS VERSION USED IN THIS DOCUMENT**<br><br>The instructions, configuration settings, and hyperlink references contained in this document are based on **CAS 5.2.7**. Most of the information presented here is also applicable to the CAS 5.1.*x* and CAS 5.3.*x* branches, with the caveat that some features may not exist in earlier versions, and the names or values of some configuration settings may have changed between releases.<br><br>The best way to learn about changes from one release to another is to read Misagh Moayyed's \"Changelog\" blog posts. These come out with each release candidate, and describe the significant changes from the previous release candidate (cumulatively, the changelogs for the several release candidates describe the changes from one feature release to the next). To access these blog posts, go to the [Releases][casrepo-releases] section of the [CAS GitHub Repository][casrepo] and search for \"Changelog\"." %}

{% include warning.html content="The instructions here *will not* work for building, configuring, or deploying CAS 6." %}

[Apereo CAS 5][casdocs] was released in November 2016. The new release improved on many of the enhancements introduced in the CAS 4 series of releases, and also introduced several new features that will enable The New School to offer an improved single sign-on experience to its users.

This document provides step-by-step instructions for setting up Apereo CAS 5. It was created to record the configuration choices made, and deployment lessons learned, during the process of building a brand new development environment to experiment with many of CAS 5's new features.

The New School's major implementation goals for this environment are:

* Apply lessons learned from our CAS 3.5 environment
    * "If we had it to do over again, we'd do this differently." This is our chance.
* High availability (fault-tolerant) everything
    * Main CAS servers reside behind load balancers
        * Servers can be rebooted, taken down for upgrade, additional servers can be added, etc., all transparently to the users.
    * Spring Cloud Configuration server
        * Allows configurations for development, test, and production to be maintained in the same location and distributed across all servers.
    * Distributed service registry
        * Keeps registered services synchronized across all servers.
    * Distributed ticket registry
        * Distributes tickets across all back-end servers so that any server can service a client, even when servers go down or restart.
* Support for additional protocols
    * Built-in support for SAML 2.0 IdP and SP
        * No more Shibboleth servers!
        * Support for many SPs built in: Adobe Creative Cloud, Google Apps, Office 365, Tableau, Workday, ...
    * Built-in support for multi-factor authentication
        * Duo Security (forthcoming for faculty and staff)
        * Google Authenticator (perhaps, for students)
* New management console and services management webapps
    * Management console
        * Dashboard for monitoring server status and performance
        * Active sessions and authentications
        * Metrics and statistics
    * Services management
        * Add, edit, delete, enable, disable services
        * Attribute release, access rules, etc.
* Other interesting features (for experimentation)
    * Risk-based authentications
    * Password management

Although the Apereo development team has dramatically simplified the configure-build-deploy process, CAS 5 is still a complex system with a lot of moving parts, and there can be a pretty steep learning curve for someone who's never done it before. Since there's not a lot of up-to-date step-by-step how-to documentation out there, we're offering what we've learned in the hope that others will find it helpful.

{% include callout.html type="warning" content="**DISCLAIMER**<br><br>The instructions and settings provided in this document may not be the only way to do things. They may not be the best way to do things. They may not even be the right way to do things. They work for us, but they may not work for you. You should carefully evaluate every suggestion, recommendation, and instruction in the context of your environment and decide whether or not it makes sense. Make sure you know and understand what's going to happen ***before*** you press the \"Enter\" key. When in doubt, [Read The Fine Manual][casdocs].<br><br>No warranty express or implied. May cause drowsiness. Your mileage may vary. Not intended to diagnose, treat, cure, or prevent any disease. Professional driver on closed course. Safety goggles recommended. Use with adult supervision. Keep out of reach of children. Do not eat." %}

{% unless site.output == "pdf" %}
## PDF version

  A PDF version of this document may be downloaded for off-line reading.

  <a target="\_blank" class="noCrossRef" href="pdf/deploying-apereo-cas.pdf">
    <button type="button" class="btn btn-default" aria-label="Left Align">
      <span class="glyphicon glyphicon-download-alt" aria-hidden="true"></span>
      PDF Download
    </button>
  </a>
{% endunless %}

{% include reflinks.md %}
