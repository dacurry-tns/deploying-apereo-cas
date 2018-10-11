---
title: Update other relevant views
last_updated: October 10, 2018
sidebar: main_sidebar
permalink: ui_develop_update-other-relevant-views.html
summary:
---

Once the layout template and the login view have been customized, most of the difficult user interface customization work is done. However, there are a number of other views that should be customized to maintain a consistent look and feel. At a minimum, these views should be modified to use the custom Thymeleaf layout template instead of the default:

```html
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{newschool/layout}">
```

Depending on the particular view, it may also be desirable to update some of the text displayed in the view by redefining the relevant properties in `WEB-INF/classes/custom_messages.properties`.

## Views that may need to be customized

Some of the likely candidates in `templates/newschool` for customization include:

* **Generic views.** CAS provides a "generic success" view (`casGenericSuccessView.html`) that is displayed when a user logs directly into the CAS server without having been directed there by another service (much like we did when first building the server). It also provides a "service error" view (`casServiceErrorView.html`) that is displayed when there is an error with the service (usually an attempt to use a service that is not in the service registry).
* **MFA views.** CAS provides one or more views for each multi-factor authentication product that it supports: `casDuoLoginView.html`, `casGoogleAuthenticatorLoginView.html`, `casGoogleAuthenticatorRegistrationView.html`, etc. Since these become part of the login web flow, they should have the same look and feel as the "base" login page.
* **LPEE views.** CAS provides views for the various LDAP Password Policy Enforcement (LPPE) outcomes: `casAccountDisabledView.html`, `casAccountLockedView.html`, `casExpiredPassView.html`, etc. These become part of the login web flow if LPPE support is enabled.
* **Password management views.** CAS provides several views associated with its (optional) user password management features: `casMustChangePassView.html`, `casPasswordUpdateSuccessView.html`, `casResetPasswordSendInstructionsView.html`, etc. These may also become part of the login web flow.

There are a variety of other views included with CAS that are not mentioned above; generally, any feature that will result in interaction with a user will have a view (or multiple views) associated with it. The files containing the views are, for the most part, named in a manner that should make it obvious which feature they belong to. It's only necessary to update those views that belong to enabled features.

## Error views

The `templates/newschool/error.html` view is used to display a variety of error messages that may occur during interaction with the user; this view should be customized as appropriate.

The views in the `templates/newschool/error` subdirectory (`401.html`, `403.html`, `404.html`, `405.html`, and `423.html`) are displayed by Tomcat when the associated HTTP error occurs; these should be customized with at least the custom layout template.

## Dashboard views

The `templates/newschool/monitoring` subdirectory contains the views associated with the CAS dashboard (admin pages). These pages come with their own layout template (`templates/newschool/monitoring/layout.html`); they do not use the same template as the "user" views. It's not necessary to customize these views, but it may be desirable depending on the audience that will be using them.

{% include reflinks.md %}
{% include links.html %}
