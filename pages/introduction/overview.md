---
title: Introduction
last_updated: August 23, 2017
sidebar: main_sidebar
permalink: introduction_overview.html
summary: This document provides step-by-step instructions for setting up an Apereo CAS 5 environment. It was created during the process of building a brand new development environment to experiment with many of the new features in this release.
---

[Apereo CAS 5][casdocs] was released in November 2016. The new release improved on many of the enhancements introduced in the CAS 4 series of releases, and also introduced several new features that will enable The New School to offer an improved single sign-on experience to its users. This document provides step-by-step instructions for setting up Apereo CAS 5. It was created to record the configuration choices made, and deployment lessons learned, during the process of building a brand new development environment to experiment with many of CAS 5's new features.

The New School's major implementation goals for this environment are:

* Apply lessons learned from our CAS 3.5 environment
    * "If we had it to do over again, we'd do this differently." This is our chance.
* High availability (fault-tolerant) everything
    * Main CAS servers reside behind load balancers
        * Servers can be rebooted, taken down for upgrade, additional servers can be added, etc., all transparently to the users
    * Configuration server
        * Allows configurations for development, test, and production to be maintained in the same location and distributed across all servers
    * Services registry
        * Keeps registered services synchronized across all servers
    * Ticket registry
        * Distributes tickets across all back-end servers so that any server can service a client, even when servers go down or restart
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

## PDF version

{% unless site.output == "pdf" %}
  A PDF version of this document may be downloaded for off-line reading.

  <a target="_blank" class="noCrossRef" href="pdf/deploying-apereo-cas.pdf">
    <button type="button" class="btn btn-default" aria-label="Left Align">
      <span class="glyphicon glyphicon-download-alt" aria-hidden="true"></span>
      PDF Download
    </button>
  </a>
{% endunless %}

{% include reflinks.md %}
