---
title: Update the logout view
last_updated: October 10, 2018
sidebar: main_sidebar
permalink: ui_develop_update-the-logout-view.html
summary:
---

The logout view is displayed when a user logs out of the CAS service, i.e., when his or her browser is directed to the `/cas/logout` endpoint. Unlike the default login view, the default logout view (`templates/casLogoutView.html`) does not include any fragments, it just displays some message text:

```html
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{layout}">

<head>
  <title th:text="#{screen.logout.header}"></title>
</head>

<body id="cas">
<div layout:fragment="content">
  <div class="alert alert-success">
    <h2 th:utext="#{screen.logout.header}"/>
    <p th:utext="#{screen.logout.success}"/>
    <p th:utext="#{screen.logout.security}" />
  </div>
</div>
</body>
</html>
```

## Updating the logout view template

For our custom logout view (`templates/newschool/casLogoutView.html`), the only change we really need to make is to replace the default layout template with our custom template:

```html
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{newschool/layout}">
```

Although not strictly necessary, we will also change a couple of the text strings to better match our local environment. We can do this by overriding their values in `WEB-INF/classes/custom_messages.properties`:

```properties
screen.logout.success=You have successfully logged out of the New School \
  Single Sign-On Service. You may <a href="login">log in</a> again.
screen.logout.security=For security reasons, please exit your web browser.
```

{% include reflinks.md %}
{% include links.html %}
