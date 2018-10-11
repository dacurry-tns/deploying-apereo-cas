---
title: Add a new theme to the overlay
last_updated: October 11, 2018
sidebar: main_sidebar
permalink: ui_add-a-new-theme-to-the-overlay.html
summary:
---

The New School CAS login page is different enough from the CAS default page that we will need to provide both custom decorative elements (styles and scripts) and custom structural elements (HTML views). The files associated with these new elements will be incorporated into the CAS WAR file using the Maven overlay.

## Create the Maven `src`  directory

To include original source material in the CAS WAR file created by the Maven build process, that material should be placed in the `src` directory of the Maven project. Maven uses the `src` directory and various subdirectories therein to organize the source material and determine when it should be used and where it should be copied to (if anywhere). The reference documentation, linked at the bottom of this page, provides more detail about this.

For the purposes of including the files associated with our new theme, we will use the `src/main/resources` directory. Maven will copy the contents of this directory into the WAR file under `WEB-INF/classes` (i.e., `src/main/resources/foo` will appear in the WAR file as `WEB-INF/classes/foo`). This is the "overlay" part of our Maven WAR overlay project&mdash;files and subdirectories with unique names will be added to the WAR file, while files and subdirectories with conflicting names will replace (overwrite) the originals.

Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# mkdir -p src/main/resources
```

on the master build server (***casdev-master***) to create this directory.

## Define the `newschool` theme

To define the `newschool` theme, first create the file `src/main/resources/newschool.properties` (recall that the contents of `src/main/resources` will be overlaid onto `WEB-INF/classes`). This file will specify the decorative elements (styles and scripts) associated with the new theme:

```properties
standard.custom.css.file:   /themes/newschool/css/newschool.css
admin.custom.css.file:      /themes/newschool/css/admin.css
cas.javascript.file:        /themes/newschool/js/newschool.js
```

The files may be named any way that makes sense; they do not have to be named `cas.css`, `cas.js`, etc. Here we have chosen to use the name of the theme to make it clear when we include them what they are. Next, create the [directory structure that will hold the theme's decorative elements][ui_how-cas-themes-work.html#changing-decorative-elements-styles-and-scripts] by running the commands

```console
casdev-master# cd src/main/resources
casdev-master# for sub in css js images fonts
> do
> mkdir -p static/themes/newschool/${sub}
> done
casdev-master#  
```

These are the subdirectories that will hold the theme's CSS style sheets, JavaScript files, images, and fonts.

### Copy in theme files from the mock-up

We already created many of the files that will reside in these subdirectories when we created the mock-up website (see the HTML shown in the [overview][ui_overview]). So, rather than create them again from scratch, we will copy them from there:

```console
casdev-master# cd static/themes/newschool
```

Copy the CSS files (we have chosen to merge `newschool.css` and `testweb.css` into a single file):

```console
casdev-master# curl -L https://testweb.newschool.edu/sso/css/newschool.css https://testweb.newschool.edu/sso/css/tnsfonts.css > newschool.css
casdev-master# curl -L https://testweb.newschool.edu/sso/css/bootstrap-material-design.min.css -o css/bootstrap-material-design.min.css
casdev-master# curl -L https://testweb.newschool.edu/sso/css/ripples.min.css -o css/ripples.min.css
casdev-master# touch css/admin.css
```

We create an empty `admin.css` file, since it's referenced in `newschool.properties` above but was not part of the mock-up website. Next, copy the JavaScript files:

```console
casdev-master# curl -L https://testweb.newschool.edu/sso/js/material.min.js -o js/material.min.js
casdev-master# curl -L https://testweb.newschool.edu/sso/js/ripples.min.js -o js/ripples.min.js
casdev-master# touch js/newschool.js
```

We create an empty `newschool.js` file, since it's referenced in `newschool.properties` above but was not part of the mock-up website. Next, copy the image files:

```console
casdev-master# curl -L https://testweb.newschool.edu/sso/images/background.jpg -o images/background.jpg
casdev-master# curl -L https://www.newschool.edu/favicon.ico -o images/favicon.ico
casdev-master# curl -L https://www.newschool.edu/framework/imgs/tns-appletouch-icon.png -o images/appleicon.png
```

The mock-up website pulled these files from difference sources; we've collected them all into the `images` subdirectory for the CAS server. Finally, copy the font files:

```console
casdev-master# wget -q -np -nH -R '*html*' --cut-dirs=1 -r https://testweb.newschool.edu/sso/fonts/
```

There are, of course, other ways these files can be copied into the overlay, or they can be created from scratch if there is no mock-up website to work from.

## Create the `newschool` template set

Rather than creating an entire new set of HTML views (Thymeleaf templates) from scratch, we well start with a copy of the default views and customize them.

The easiest way to get a copy of the default templates is to simply copy them from the deployed application directory . Run the commands

```console
casdev-master# cd /opt/workspace/cas-overlay-template
casdev-master# cd src/main/resources
casdev-master# mkdir templates
casdev-master# cp -rp /var/lib/tomcat/cas/WEB-INF/classes/templates templates/newschool
casdev-master# chown -R root.root templates/newschool
casdev-master# chmod -R og+rX templates/newschool
```

to copy the default templates from the deployed application directory to the [directory structure that will hold the theme's structural elements][ui_how-cas-themes-work.html#changing-structural-elements-html-views].

## The complete theme directory structure

Once all the commands above have been executed, the `src/main/resources` directory in the overlay should look like this:

```console
src
└── main/
    └── resources/
        ├── custom_messages.properties
        ├── newschool.properties
        ├── static/
        │   └── themes/
        │       └── newschool/
        │           ├── css/
        │           │   ├── admin.css
        │           │   ├── bootstrap-material-design.min.css
        │           │   ├── newschool.css
        │           │   └── ripples.min.css
        │           ├── fonts/
        │           │   └── Neue/
        │           │       ├── Neue-Black.eot
        │           │       ├── Neue-Black.svg
        │           │       ├── Neue-Black.ttf
        │           │       ├── Neue-Black.woff
        │           │       ├── Neue-Bold.eot
        │           │       ├── Neue-Bold.svg
        │           │       ├── Neue-Bold.ttf
        │           │       ├── Neue-Bold.woff
        │           │       ├── Neue-BoldItalic.svg
        │           │       ├── Neue-BoldItalic.ttf
        │           │       ├── Neue-BoldItalic.woff
        │           │       ├── Neue-Regular.eot
        │           │       ├── Neue-Regular.svg
        │           │       ├── Neue-Regular.ttf
        │           │       ├── Neue-Regular.woff
        │           │       ├── Neue-RegularItalic.eot
        │           │       ├── Neue-RegularItalic.svg
        │           │       ├── Neue-RegularItalic.ttf
        │           │       ├── Neue-RegularItalic.woff
        │           │       ├── NeueDisplay-Black.eot
        │           │       ├── NeueDisplay-Black.svg
        │           │       ├── NeueDisplay-Black.ttf
        │           │       ├── NeueDisplay-Black.woff
        │           │       ├── NeueDisplay-Random.eot
        │           │       ├── NeueDisplay-Random.svg
        │           │       ├── NeueDisplay-Random.ttf
        │           │       ├── NeueDisplay-Random.woff
        │           │       ├── NeueDisplay-Ultra.eot
        │           │       ├── NeueDisplay-Ultra.svg
        │           │       ├── NeueDisplay-Ultra.ttf
        │           │       ├── NeueDisplay-Ultra.woff
        │           │       ├── NeueDisplay-Wide.eot
        │           │       ├── NeueDisplay-Wide.svg
        │           │       ├── NeueDisplay-Wide.ttf
        │           │       └── NeueDisplay-Wide.woff
        │           ├── images/
        │           │   ├── appleicon.png
        │           │   ├── background.jpg
        │           │   └── favicon.ico
        │           └── js/
        │               ├── material.min.js
        │               ├── newschool.js
        │               └── ripples.min.js
        └── templates/
            └── newschool/
                ├── casAcceptableUsagePolicyView.html
                ├── casAccountDisabledView.html
                ├── casAccountLockedView.html
                ├── casAuthenticationBlockedView.html
                ├── casAuthyLoginView.html
                ├── casAzureAuthenticatorLoginView.html
                ├── casBadHoursView.html
                ├── casBadWorkstationView.html
                ├── casConfirmLogoutView.html
                ├── casConfirmView.html
                ├── casConsentLogoutView.html
                ├── casConsentReviewView.html
                ├── casConsentView.html
                ├── casDuoLoginView.html
                ├── casExpiredPassView.html
                ├── casGenericSuccessView.html
                ├── casGoogleAuthenticatorLoginView.html
                ├── casGoogleAuthenticatorRegistrationView.html
                ├── casGuaDisplayUserGraphicsView.html
                ├── casGuaGetUserIdView.html
                ├── casInterruptView.html
                ├── casLoginMessageView.html
                ├── casLoginView.html
                ├── casLogoutView.html
                ├── casMfaRegisterDeviceView.html
                ├── casMustChangePassView.html
                ├── casPac4jStopWebflow.html
                ├── casPasswordUpdateSuccessView.html
                ├── casPropagateLogoutView.html
                ├── casRadiusLoginView.html
                ├── casResetPasswordErrorView.html
                ├── casResetPasswordSendInstructionsView.html
                ├── casResetPasswordSentInstructionsView.html
                ├── casResetPasswordVerifyQuestionsView.html
                ├── casRiskAuthenticationBlockedView.html
                ├── casServiceErrorView.html
                ├── casSurrogateAuthnListView.html
                ├── casSwivelLoginView.html
                ├── casU2fLoginView.html
                ├── casU2fRegistrationView.html
                ├── casYubiKeyLoginView.html
                ├── casYubiKeyRegistrationView.html
                ├── error/
                │   ├── 401.html
                │   ├── 403.html
                │   ├── 404.html
                │   ├── 405.html
                │   └── 423.html
                ├── error.html
                ├── fragments/
                │   ├── bottom.html
                │   ├── cas-resources-list.html
                │   ├── cookies.html
                │   ├── defaultauthn.html
                │   ├── footer.html
                │   ├── footerButtons.html
                │   ├── head.html
                │   ├── insecure.html
                │   ├── loginProviders.html
                │   ├── loginform.html
                │   ├── loginsidebar.html
                │   ├── logo.html
                │   ├── modal.html
                │   ├── pwdupdateform.html
                │   ├── serviceui.html
                │   └── top.html
                ├── layout.html
                ├── monitoring/
                │   ├── attrresolution.html
                │   ├── layout.html
                │   ├── viewAuthenticationEvents.html
                │   ├── viewConfig.html
                │   ├── viewConfigMetadata.html
                │   ├── viewDashboard.html
                │   ├── viewLoggingConfig.html
                │   ├── viewSsoSessions.html
                │   ├── viewStatistics.html
                │   └── viewTrustedDevices.html
                └── protocol/
                    ├── 2.0/
                    │   ├── casProxyFailureView.html
                    │   ├── casProxySuccessView.html
                    │   ├── casServiceValidationFailure.html
                    │   └── casServiceValidationSuccess.html
                    ├── 3.0/
                    │   ├── casServiceValidationFailure.html
                    │   └── casServiceValidationSuccess.html
                    ├── casPostResponseView.html
                    ├── oauth/
                    │   └── confirm.html
                    ├── oidc/
                    │   └── confirm.html
                    └── openid/
                        ├── casOpenIdAssociationSuccessView.html
                        ├── casOpenIdServiceFailureView.html
                        ├── casOpenIdServiceSuccessView.html
                        └── user.html
```

The following pages in this section will discuss how to customize these pages.

## References

* [Apache Maven: Introduction to the Standard Directory Layout][maven-dir-layout]

{% include reflinks.md %}
{% include links.html %}
