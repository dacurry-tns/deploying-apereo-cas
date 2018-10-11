---
title: Update the login view
last_updated: October 10, 2018
sidebar: main_sidebar
permalink: ui_develop_update-the-login-view.html
summary:
---

The CAS login view is built from a content template (`casLoginView.html`), a fragment (`fragments/loginform.html`), and some message strings defined in `messages.properties` (or `messages_xx.properties` or `custom_messages.properties`). When the content template is substituted into the Thymeleaf layout template (`layout.html`), the result will be the CAS login page that is displayed to the user.

## Updating the login view content template

The default login view (`templates/casLoginView.html`) defines two columns on the page: the left-hand column contains the actual login dialog box, and the right-hand column contains a number of "additional information" boxes. All of these elements are defined by different fragments.

```html
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{layout}">

<head>
  <title th:text="#{cas.login.pagetitle}"></title>
</head>

<body id="cas" class="login">
  <div layout:fragment="content">
    <div class="row">
      <div id="notices" class="col-sm-12 col-md-6 col-md-push-6">
        <div th:replace="fragments/insecure"/>
        <div th:replace="fragments/defaultauthn"/>
        <div th:replace="fragments/cookies"/>
        <div th:replace="fragments/serviceui"/>
        <div th:replace="fragments/cas-resources-list" />
        <div th:replace="fragments/loginProviders" />
      </div>
      <div class="col-sm-12 col-md-6 col-md-pull-6">
        <div th:replace="fragments/loginform" />
      </div>
    </div>
  </div>
</body>
</html>
```

As a reminder, this looks something like this (not all of the fragments in the right-hand column are displayed by default):

{% include image.html file="ui/develop/fig26-default-cas-login-page.png" alt="Browser Screen Shot" caption="Figure 26. The default CAS server login page" %}

Our login view (`templates/newschool/casLoginView.html`) will be a little simpler, with just one fragment, the login dialog box:

```html
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{newschool/layout}">

<head>
  <title th:text="#{cas.login.pagetitle}"></title>
</head>

<body id="cas" class="login">
  <div layout:fragment="content">
    <div class="row">
      <div class="col-sm-12 col-md-4 col-md-offset-4">
        <div th:replace="newschool/fragments/loginform"/>
      </div>
    </div>
  </div>
</body>
</html>
```

As explained previously, Thymeleaf interprets all relative file paths as if they were rooted at the `templates` directory. Therefore, aside from removing the extra fragments that we're not going to use, we have to make two other changes in our template:

1. In the `layout:decorate` attribute of the `<html>` tag, we will replace `~{layout}` with `~{newschool/layout}`. If we do not make this change, our content template will be applied to the default layout template (`templates/layout.html`) rather than our custom layout (`templates/newschool/layout.html`).
2. In the `th:replace` attribute of the inner-most `<div>` tag, we will replace `fragments/loginform` with `newschool/fragments/loginform`. If we did not make this change, the default login form fragment would be included instead of our customized version.

## Updating the login form fragment

To create our login form fragment (`templates/newschool/fragments/loginform.html`), we will start with the "login form box" section of the mock-up page. From there, we will replace "hard coded" text strings with references to message property names, and incorporate any "interesting" or "desirable" features from the default login form fragment or one of the other fragments that our login view is not including. The result will be something like this:

```html
<div class="well" id="login">
  <header class="tns-header">
    <div class="tns-banner">
      <h2 class="tns-sitename">Single Sign-On</h2>
    </div>
  </header>

  <h1 th:text="#{cas.login.pagetitle}"></h1>

  <div class="registered-service" th:if="${registeredService}">
    <div th:if="${serviceUIMetadata}">
      <p>to continue to
         <span th:text="${serviceUIMetadata.displayName}"></span></p>
    </div>
    <div th:unless="${serviceUIMetadata}">
      <p>to continue to
        <span th:text="${registeredService.name}"></span></p>
    </div>
  </div> <!-- registered-service -->

  <form method="post" id="fm1" th:object="${credential}" action="login">
    <div class="alert-row">
      <div class="alert alert-danger" th:if="${#fields.hasErrors('*')}">
        <span th:each="err : ${#fields.errors('*')}" th:utext="${err}"/>
      </div>
    </div> <!-- alert-row -->

    <div class="form-group label-floating is-empty">
      <label class="control-label" for="username"
        th:utext="#{screen.welcome.label.netid}"/>
      <div th:if="${openIdLocalId}">
        <strong>
          <span th:utext="${openIdLocalId}"/>
        </strong>
        <input type="hidden"
               id="username"
               name="username"
               th:value="${openIdLocalId}"/>
      </div>
      <div th:unless="${openIdLocalId}">
        <input class="required form-control"
               id="username"
               name="username"
               size="25"
               tabindex="1"
               type="text"
               th:disabled="${guaEnabled}"
               th:field="*{username}"
               th:accesskey="#{screen.welcome.label.netid.accesskey}"
               autocomplete="off"/>
      </div>
    </div> <!-- form-group -->

    <div class="form-group label-floating is-empty" style="padding-bottom: 0px">
      <label class="control-label" for="password"
        th:utext="#{screen.welcome.label.password}"/>
      <div>
        <input class="required form-control"
               type="password"
               id="password"
               name="password"
               size="25"
               tabindex="2"
               th:accesskey="#{screen.welcome.label.password.accesskey}"
               th:field="*{password}"
               autocomplete="off"/>
        <div class="capslock-msg">
          <p id="capslock-on" style="display: none">
            <i class="material-icons">error</i>
            <span th:utext="#{screen.capslock.on}"/>
          </p>
        </div>
      </div>
    </div> <!-- form-group -->

    <div class="form-row continue-button" th:if="${recaptchaSiteKey}">
      <div class="g-recaptcha" th:attr="data-sitekey=${recaptchaSiteKey}"/>
    </div> <!-- form-row -->
    <div class="form-row text-center">
      <input type="hidden" name="execution" th:value="${flowExecutionKey}"/>
      <input type="hidden" name="_eventId" value="submit"/>
      <input type="hidden" name="geolocation"/>

      <input class="btn btn-primary btn-lg" style="margin-top: 0px"
             name="submit"
             accesskey="l"
             th:value="#{screen.welcome.button.login}"
             tabindex="6"
             type="submit"/>
      <div class="ripple-container"></div>
    </div> <!-- form-row -->

    <div class="form-row account-options">
      <p>
        <a th:href="#{screen.welcome.resetPassword.url}">
          <i class="material-icons">lock</i>
          <span th:utext="#{screen.welcome.resetPassword.text}"/>
        </a>
      </p>
      <p>
        <a th:href="#{screen.welcome.lookupNetID.url}">
          <i class="material-icons">help</i>
          <span th:utext="#{screen.welcome.lookupNetID.text}"/>
        </a>
      </p>
    </div> <!-- form-row -->

    <div class="form-row text-right privacy-terms">
      <a th:href="#{screen.welcome.privacy.url}" target="_blank">
        <span th:utext="#{screen.welcome.privacy.text}"/>
      </a>
      <a th:href="#{screen.welcome.terms.url}" target="_blank">
        <span th:utext="#{screen.welcome.terms.text}"/>
      </a>
    </div> <!-- form-row -->
  </form>
</div>
```

Some of the changes that will be made from the mock-up include:

* **Line 7 (`<h1>` tag).** Replace the hard coded "Log in" with a reference to a message property.
* **Lines 10-19 (`registered-service` class).** Incorporate some Thymeleaf code from the default `serviceui` fragment that obtains the name of the service (as defined in the service registry) the user is attempting to access. The `if`/`unless` sequence tests to see if the service is a SAML2-based service or a CAS-based service and obtains the name from the appropriate variable.
* **Lines 22-26 (`alert-row` class).** Incorporate some Thymeleaf code from the default `loginform` fragment that displays any form input error messages.
* **Lines 28-74 (username and password prompts).** Include the more complex username and password prompt code from the default `loginform` fragment. This includes support for OpenID, caps lock detection, etc.
* **Lines 93-115 (links for password reset, NetID lookup, privacy, and terms).** Replace the hard coded text for these links, as well as the links themselves, with message property references.

## Updating the text strings

As noted above, the login form fragment uses message properties for all of its text, rather than hard coded values. Some of these properties are defined by the default CAS message bundles (`messages.properties` for the `en` locale and `messages_xx.properties` for other locales). We need to change the values of some of these properties to better match our environment (for example, we prefer the term "NetID" to "Username"). Other properties are specific to our login form and are not included in the default message bundles; we will have to define those properties ourselves. We can use the same file to accomplish both of these tasks; properties defined in the `WEB-INF/classes/custom_messages.properties` file will override properties of the same name defined in `messages.properties` or `messages_xx.properties`. So, our `custom_messages.properties` file will look something like this:

```properties
# Just in case (we don't display this currently)
copyright=Copyright &copy; 2018 The New School

# Page title
cas.login.pagetitle=Log in

# Replace "Username" with "NetID"
screen.welcome.label.netid=<span class="accesskey">N</span>etID:
screen.welcome.label.netid.accesskey=n

# Replace "LOGIN" with "Continue"
screen.welcome.button.login=Continue

# Text and url for password reset
screen.welcome.resetPassword.url=https://account.newschool.edu/cgi-bin/acctservices.pl?f=i&s=pr
screen.welcome.resetPassword.text=Reset your password

# Text and url for NetID lookup
screen.welcome.lookupNetID.url=https://account.newschool.edu/cgi-bin/acctservices.pl?f=i&s=gn
screen.welcome.lookupNetID.text=Look up your NetID

# Text and url for the "privacy" link
screen.welcome.privacy.url=//www.newschool.edu/privacy-policy/
screen.welcome.privacy.text=Privacy

# Text and url for the "terms" link
screen.welcome.terms.url=//it.newschool.edu/sites/default/files/uploads/documents/Statement%20on%20the%20Responsibilities%20of%20Computer%20Users%20v2.1.pdf
screen.welcome.terms.text=Terms

# Text to display when caps lock is on
screen.capslock.on=Caps Lock is on
```

{% include reflinks.md %}
{% include links.html %}
