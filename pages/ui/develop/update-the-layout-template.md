---
title: Update the layout template
last_updated: October 10, 2018
sidebar: main_sidebar
permalink: ui_develop_update-the-layout-template.html
summary:
---

In general, the layout template is responsible for providing most of the `<head>` element content across all the views, so that they all look the same. And it's also responsible for providing the common-to-all-views components of the `<body>` element (headers, footers, and such). The main tasks in updating the layout template, therefore, are merging the `<head>` and `<body>` elements we created in the mock-up login page with the ones provided by the CAS project in `layout.html`.

### Merging the `<head>` elements

The mock-up login page includes a number of external style sheets, scripts, and fonts:

```html
<head>
  <meta charset="UTF-8"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>

  <title>Log in - New School SSO</title>

  <!-- JQuery -->
  <script src="//code.jquery.com/jquery-1.10.2.min.js"></script>

  <!-- Bootstrap -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  <script src="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>

  <!-- Material Design for Bootstrap -->
  <link rel="stylesheet" href="css/bootstrap-material-design.min.css">
  <link rel="stylesheet" href="css/ripples.min.css">
  <script src="js/material.min.js"></script>
  <script src="js/ripples.min.js"></script>

  <!-- TNS icons -->
  <link rel="icon" type="image/x-icon" href="//www.newschool.edu/favicon.ico">
  <link rel="apple-touch-icon" href="//www.newschool.edu/framework/imgs/tns-appletouch-icon.png">

  <!-- Material Design fonts -->
  <link rel="stylesheet" href="//fonts.googleapis.com/css?family=Roboto:300,400,500,700">
  <link rel="stylesheet" href="//fonts.googleapis.com/icon?family=Material+Icons">

  <!-- Page-specifc styles -->
  <link rel="stylesheet" href="css/tnsfonts.css">
  <link rel="stylesheet" href="css/newschool.css">
</head>
```

The default layout template (`templates/layout.html`) provides some of the same style sheets and scripts (e.g., jQuery, Bootstrap), but it provides them locally through *webjars* instead of lining to content distribution servers on the Internet. It also includes some additional scripts that are used by CAS features (password strength meter, geolocation, etc.):

```html
<head>
  <meta charset="UTF-8"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>

  <title layout:title-pattern="$CONTENT_TITLE - $LAYOUT_TITLE">CAS &#8211; Central Authentication Service</title>

  <link rel="stylesheet" th:href="@{#{webjars.fontawesomemin.css}}"/>
  <link type="text/css" rel="stylesheet" th:href="@{#{webjars.bootstrapmin.css}}"/>
  <link type="text/css" rel="stylesheet" th:href="@{#{webjars.latomin.css}}"/>

  <link rel="stylesheet" th:href="@{${#themes.code('standard.custom.css.file')}}"/>

  <link rel="icon" th:href="@{/favicon.ico}" type="image/x-icon"/>

  <script type="text/javascript" th:src="@{#{webjars.zxcvbn.js}}"></script>
  <script type="text/javascript" th:src="@{#{webjars.jquerymin.js}}"></script>
  <script type="text/javascript" th:src="@{#{webjars.jqueryui.js}}"></script>
  <script type="text/javascript" th:src="@{#{webjars.jquerycookie.js}}"></script>
  <script src="//www.google.com/recaptcha/api.js" async defer th:if="${recaptchaSiteKey}"></script>
  <script th:src="@{#{webjars.bootstrapmin.js}}"></script>

  <script th:inline="javascript">
    /*<![CDATA[*/

    var trackGeoLocation = /*[[${trackGeoLocation}]]*/ === "true";

    var googleAnalyticsTrackingId = /*[[${googleAnalyticsTrackingId}]]*/;

    if (googleAnalyticsTrackingId != null && googleAnalyticsTrackingId != '') {
        (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
                    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
                m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
        })(window, document, 'script', 'https://www.google-analytics.com/analytics.js', 'ga');

        ga('create', googleAnalyticsTrackingId, 'auto');
        ga('send', 'pageview');
    }

    /*]]>*/
  </script>
</head>
```

The goal then, is to merge these two `<head>` elements together, along with some other appropriate changes, and put them all together in our customized layout template (`templates/newschool/layout.html`):

```html
<head>
  <meta charset="UTF-8"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>

  <title layout:title-pattern="$CONTENT_TITLE - $LAYOUT_TITLE">
    New School SSO
  </title>

  <!-- JQuery -->
  <script type="text/javascript" th:src="@{#{webjars.jquerymin.js}}"></script>
  <script type="text/javascript" th:src="@{#{webjars.jqueryui.js}}"></script>
  <script type="text/javascript" th:src="@{#{webjars.jquerycookie.js}}"></script>

  <!-- Bootstrap -->
  <link rel="stylesheet" type="text/css" th:href="@{#{webjars.bootstrapmin.css}}">
  <script th:src="@{#{webjars.bootstrapmin.js}}"></script>

  <!-- Material Design for Bootstrap -->
  <link rel="stylesheet" th:href="@{${#themes.code('md4b.css.file')}}">
  <link rel="stylesheet" th:href="@{${#themes.code('ripples.css.file')}}">
  <script th:src="@{${#themes.code('md4b.js.file')}}"></script>
  <script th:src="@{${#themes.code('ripples.js.file')}}"></script>

  <!-- Material Design fonts -->
  <link rel="stylesheet" href="//fonts.googleapis.com/css?family=Roboto:300,400,500,700">
  <link rel="stylesheet" href="//fonts.googleapis.com/icon?family=Material+Icons">

  <!-- 'newschool' theme -->
  <link rel="stylesheet" th:href="@{${#themes.code('standard.custom.css.file')}}">

  <!-- TNS icons -->
  <link rel="icon" type="image/x-icon" th:href="@{${#themes.code('favicon.img.file')}}">
  <link rel="icon" type="image/png" th:href="@{${#themes.code('appleicon.img.file')}}">

  <script th:if="${recaptchaSiteKey}" async defer
    src="//www.google.com/recaptcha/api.js"></script>

  <script th:inline="javascript">
    /*<![CDATA[*/

    var trackGeoLocation = /*[[${trackGeoLocation}]]*/ === "true";
    var googleAnalyticsTrackingId = /*[[${googleAnalyticsTrackingId}]]*/;

    if (googleAnalyticsTrackingId != null && googleAnalyticsTrackingId != '') {
      (function(i, s, o, g, r, a, m) {
        i['GoogleAnalyticsObject'] = r;
        i[r] = i[r] || function() {
          (i[r].q = i[r].q || []).push(arguments)
        }, i[r].l = 1 * new Date();
        a = s.createElement(o),
          m = s.getElementsByTagName(o)[0];
        a.async = 1;
        a.src = g;
        m.parentNode.insertBefore(a, m)
      })(window, document, 'script', 'https://www.google-analytics.com/analytics.js', 'ga');

      ga('create', googleAnalyticsTrackingId, 'auto');
      ga('send', 'pageview');
    }

    /*]]>*/
  </script>
</head>
```

- **Lines 6-8.** Adopt the "dual title" format from the CAS server (explained in [How Thymeleaf layouts work][ui_how-thymeleaf-layouts-work]), but change the layout component of the title to "New School SSO."
- **Lines 10-17.** Pull in the jQuery and Bootstrap packages using the CAS-provided webjar URLs instead of the Internet content distribution server URLs we used in the mock-up.
- **Lines 19-24.** Pull in the Material Design for Bootstrap CSS and JavaScript files. But instead of hard-coding the paths to these files here as we did in the mock-up, use Thymeleaf's `#theme.code()` function to retrieve them from values defined in the theme definition file (`newschool.properties`):

```properties
md4b.css.file:              /themes/newschool/css/bootstrap-material-design.min.css
md4b.js.file:               /themes/newschool/js/material.min.js
ripples.css.file:           /themes/newschool/css/ripples.min.css
ripples.js.file:            /themes/newschool/js/ripples.min.js
```

- **Lines 26-28.** Pull in the Material Design fonts (used by the Material Design for Bootstrap package) from Google's content distribution servers.
- **Lines 30-35.** Pull in the New School cascading style sheet and the favicons, again by using Thymeleaf's `#theme.code()` function to retrieve the paths from `newschool.properties`:

```properties
standard.custom.css.file:   /themes/newschool/css/newschool.css

favicon.img.file:           /themes/newschool/images/favicon.ico
appleicon.img.file:         /themes/newschool/images/appleicon.png
```

- **Lines 37-64.** Copy the feature-specific scripts from the CAS-provided `layout.html` to the new one.

{% include note.html content="Webjars allow developers to bundle their client-side web libraries (JavaScript, CSS, etc.) as JAR files. They don't change the way the libraries themselves work, they just make it possible to manage them using dependency-management tools such as Maven and Gradle. It makes sense to incorporate the CAS-provided webjar version of a library if one exists to ensure we're getting the version that the CAS project supports. But we don't need to package our own locally-added web libraries (such as Material Design for Bootstrap) this way, because we're not managing them as external dependencies." %}

### Merging the `<body>` elements

The default layout template defines a high-level structure for all the web views. It includes a logo, some content, a footer, and a bottom:  

```html
<body>
  <div id="container" class="container">
    <div th:replace="fragments/logo"/>
    <div layout:fragment="content" id="content">
      <h1/>
      <p/>
    </div>
    <div th:replace="fragments/footer"/>
  </div>

  <div th:insert="fragments/bottom"/>
</body>
```

The content part of the page will be populated from a content template, while the other parts will be taken from re-usable fragments. Our custom layout will use a similar high-level structure:

```html
<body>
  <div class="container-fluid">
    <div th:replace="newschool/fragments/logo"/>
  </div>

  <div id="container" class="container">
    <div layout:fragment="content" id="content">
      &nbsp;
    </div>
    <div th:replace="newschool/fragments/footer"/>
  </div>

  <div th:insert="newschool/fragments/bottom"/>
</body>
```

There are two principal differences between the default layout and ours:
1. The paths to the fragments will be changed from `fragments/[fragment-name]` to `newschool/fragments/[fragment-name]`. By default, Thymeleaf interprets all relative file paths as if they were rooted at the `templates` directory. If we do not make this change, we will be including the fragments from the default layout (rooted at `templates`), not our custom layout (rooted at `templates/newschool`).
2. We will move our logo from the responsive, fixed-width container that the other content resides in into its own, fluid-width container. This is a Bootstrap-specific change that will result in the logo always being as wide as the entire browser viewport, regardless of how wide the other page content is.

### Updating the fragments

In addition to updating the content part of the page in the layout template, we need to make some changes to the re-usable fragments that it includes.

#### The logo fragment

The default logo fragment (`fragments/logo.html`) provides an Apereo logo:

```html
<header>
  <a id="logo" href="http://www.apereo.org" th:title="#{logo.title}">Apereo</a>
  <h1>Apereo Central Authentication Service (CAS)</h1>
</header>
```

Our custom logo fragment (`newschool/fragments/logo.html`) will replace this with the New School SVG logo definition from the mock-up login page:

```html
<!-- New School header logo lockup -->
<header role="banner" class="tns-header">
  <section class="tns-header-region">
    <div class="tns-lockup">
      <div class="tns-banner text-center">
        <h1 class="tns-uname">
          <!-- Generator: Adobe Illustrator 18.1.0, SVG Export Plug-In.
               SVG Version: 6.00 Build 0) -->
          <svg xmlns="http://www.w3.org/2000/svg" xml:space="preserve"
               version="1.1" x="0px" y="0px" viewBox="0 0 224.4 30.2"
               enable-background="new 0 0 224.4 30.2"
               id="tns-logo-svg">
            <g id="Layer_1">
              <g>
                <rect x="4.4" y="19.7" width="220" height="3.5"/>
                <rect x="4.4" y="26.7" width="220" height="3.5"/>
                <g>
                  <path d="M9,3.9v12.4H5V3.9H0V0.3h14v3.6H9z"/>
                  <path d="M29.7,10.2H19.2v6.1h-4v-16h4v6.1h10.5V0.3h4v16h-4V10.2z"/>
                  <path d="M35.5,16.3v-16h11.4v3.6h-7.4v2.7h6V10h-6v2.7h7.4v3.6H35.5z"/>
                  <path d="M57.7,6.1h-0.6v10.2h-4v-16h4.7l9.3,10h0.6v-10h4v16h-4.5L57.7,6.1z"/>
                  <path d="M73.5,16.3v-16h11.4v3.6h-7.4v2.7h6V10h-6v2.7h7.4v3.6H73.5z"/>
                  <path d="M85.2,0.3h4.5l3.5,12.7h0.9l3.2-12.7h7.5l3.6,12.7h0.9l3-12.7h4.5l-4.4,16h-7.1l-3.7-12.7h-1.1l-3.4,12.7h-7.1L85.2,0.3z"/>
                  <path d="M121.5,5.2c0-3.4,2.7-5.2,6.2-5.2c2,0,4.2,0.5,5.5,1.2l-0.8,3.4c-1.3-0.8-3.3-1.2-4.9-1.2c-1.2,0-2.1,0.5-2.1,1.3c0,2.6,8.2,1.1,8.2,6.5c0,2.9-2,5.4-6.3,5.4c-1.8,0-3.9-0.2-5.5-1L122,12c1.7,0.8,3.5,1.3,5.5,1.3c1.8,0,2.2-0.6,2.2-1.4C129.7,9.5,121.5,10.9,121.5,5.2z"/>
                  <path d="M146.9,16.1c-1.3,0.5-2.4,0.5-4.2,0.5c-5,0-8.3-3.9-8.3-8.3c0-4.5,3.5-8.3,8.7-8.3c1.3,0,2.8,0,4.1,0.5l-0.4,3.6c-1.4-0.5-2.4-0.5-3.6-0.5c-3,0-4.7,1.7-4.7,4.7c0,3,2,4.7,4.8,4.7c1.8,0,2.4-0.2,3.6-0.6V16.1z"/>
                  <path d="M163.2,10.2h-10.5v6.1h-4v-16h4v6.1h10.5V0.3h4v16h-4V10.2z"/>
                  <path d="M177.8,0c5.5,0,9.5,2.8,9.5,8.2c0,5.6-4.3,8.4-9.5,8.4c-5.2,0-9.5-2.8-9.5-8.4C168.2,3,172.1,0,177.8,0z M177.7,13.3c3.1,0,5.4-1.6,5.4-5.1c0-3.2-2.3-4.8-5.4-4.8s-5.4,1.6-5.4,4.9C172.4,11.7,174.6,13.3,177.7,13.3z"/>
                  <path d="M197.5,0c5.5,0,9.5,2.8,9.5,8.2c0,5.6-4.3,8.4-9.5,8.4c-5.2,0-9.5-2.8-9.5-8.4C187.9,3,191.8,0,197.5,0z M197.4,13.3c3.1,0,5.4-1.6,5.4-5.1c0-3.2-2.3-4.8-5.4-4.8s-5.4,1.6-5.4,4.9C192.1,11.7,194.3,13.3,197.4,13.3z"/>
                  <path d="M208,0.3h4v12.2h12.4v3.8H208V0.3z"/>
                </g>
              </g>
            </g>
          </svg>
        </h1> <!-- tns-uname -->
      </div> <!-- tns-banner -->
    </div> <!-- tns-lockup -->
  </section> <!-- tns-header-region -->
</header> <!-- tns-header -->
```

#### The footer fragment

The default footer fragment (`fragments/footer.html`) provides a copyright message and a "powered by" line that includes the CAS server version information (note the use of Thymeleaf text modifiers and value substitutions):

```html
<footer>
  <div id="copyright" class="container">
    <p th:utext="#{copyright}"></p>
    <p>Powered by <a href="http://www.apereo.org/cas">
      Apereo Central Authentication Service
      <span th:text="${T(org.apereo.cas.util.CasVersion).getVersion()}"></span>
      <span th:text="${T(org.apereo.cas.util.CasVersion).getDateTime()}"></span> </a>
    </p>
  </div>
</footer>
```

We don't want to provide detailed information about the server on the login page, and arguably, we shouldn't be providing a copyright message there either since we'll be fronting third-party services that aren't our intellectual property, so... we can just dispense with the footer all together by commenting it out in our custom fragment (`newschool/fragments/footer.html`):

```html
<!--
<footer>
  <div id="copyright" class="container">
    <p th:utext="#{copyright}"></p>
    <p>Powered by <a href="http://www.apereo.org/cas">
      Apereo Central Authentication Service
      <span th:text="${T(org.apereo.cas.util.CasVersion).getVersion()}"></span>
      <span th:text="${T(org.apereo.cas.util.CasVersion).getDateTime()}"></span> </a>
    </p>
  </div>
</footer>
-->
```

#### The bottom fragment

The bottom fragment, as envisioned by the CAS developers, is used to include JavaScript code that should not be executed until the entire page has loaded. They use this to check that several of the packages they need are loaded, and use a package called [HeadJS](http://headjs.com/) to load them if they're not:

```html
<script th:src="@{#{webjars.headmin.js}}"></script>
<script type="text/javascript" th:src="@{${#themes.code('cas.javascript.file')}}"></script>

<script th:inline="javascript">
head.ready(document, function () {
    if (!window.jQuery) {
        var jqueryUrl = /*[[@{#{webjars.jquerymin.js}}]]*/;
        head.load(jqueryUrl, loadjQueryUI);
    } else {
        notifyResourcesAreLoaded(resourceLoadedSuccessfully);
    }
});

function loadjQueryUI() {
        var jqueryUrl = /*[[@{#{webjars.jqueryui.js}}]]*/;
        head.load(jqueryUrl, loadjQueryCookies);
}

function loadjQueryCookies() {
        var jqueryUrl = /*[[@{#{webjars.jquerycookie.js}}]]*/;
        head.load(jqueryUrl, notifyResourcesAreLoaded(resourceLoadedSuccessfully));
}

function notifyResourcesAreLoaded(callback) {
    if (typeof callback === "function") {
        callback();
    }
}
</script>
```

For our custom bottom fragment, we will add two things:
1. We specifically include the default theme's `cas.js`, so that we don't have to duplicate those functions in our own `newschool.js` file. This approach allows us to benefit from any fixes/updates the CAS developers make to these functions; if we copied them into `newschool.js` we would not receive those updates (unless we manually checked for and applied them) .
2. We call the *Material Design for Bootstrap* library's `init()` function.

```html
<script th:src="@{#{webjars.headmin.js}}"></script>
<script th:src="@{/js/cas.js}"></script>
<script type="text/javascript" th:src="@{${#themes.code('cas.javascript.file')}}"></script>

<script th:inline="javascript">
  head.ready(document, function () {
    if (!window.jQuery) {
      var jqueryUrl = /*[[@{#{webjars.jquerymin.js}}]]*/;
      head.load(jqueryUrl, loadjQueryUI);
    } else {
      notifyResourcesAreLoaded(resourceLoadedSuccessfully);
    }
  });

  function loadjQueryUI() {
    var jqueryUrl = /*[[@{#{webjars.jqueryui.js}}]]*/;
    head.load(jqueryUrl, loadjQueryCookies);
  }

  function loadjQueryCookies() {
    var jqueryUrl = /*[[@{#{webjars.jquerycookie.js}}]]*/;
    head.load(jqueryUrl, notifyResourcesAreLoaded(resourceLoadedSuccessfully));
  }

  function notifyResourcesAreLoaded(callback) {
    if (typeof callback === "function") {
      callback();
    }
  }
</script>

<script>
  $(function () {
    $.material.init();
  });
</script>
```

{% include reflinks.md %}
{% include links.html %}
