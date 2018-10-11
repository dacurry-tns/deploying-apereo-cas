---
title: Customizing the CAS user interface
last_updated: October 10, 2018
sidebar: main_sidebar
permalink: ui_overview.html
summary: The CAS login page is the first thing users see when logging into any CAS-ified service. It should reflect the branding and style of the organization to be clearly recognizable, and should also take advantage of available features to prevent spoofing attempts.
---

Branding is important to The New School. As something that every member of our community encounters on a daily basis, the look and feel of the CAS login page is an important representation of that brand. Since 2014, the CAS 3.5.*x* login page has looked like the one shown in Figure 24. There are 15-20 background images showing various New School locations and activities; a different image is randomly selected each time the page is loaded.

{% include image.html file="ui/fig24-the-cas-35-login-page.png" alt="Browser Screen Shot" caption="Figure 24. The New School CAS 3.5 login page" %}

After almost four years the page is looking a little dated, the amount of text under the login button has grown over time to be somewhat unattractive, and although it works on mobile devices, it's not as nice an experience as it could be. As part of the upgrade from CAS 3.5.*x* to CAS 5, we will be implementing a brand new login page. The new page will be more in line with current web design practices, will be fully responsive with a mobile-first design, and will also better tie into the current branding on other New School web sites.

## Start with a mock-up

The CAS server uses a collection of HTML files, Thymeleaf templates, CSS and JavaScript files, and Java message bundles to implement the login page (and other pages). Rather than try to work with all of these components while designing the login page's look and feel, a process that required several rounds of reviews and changes, we created a simple mock-up of the page using plain old HTML, CSS, and JavaScript and hosted it independently of the CAS server. By doing the design outside the CAS server, we were able to focus on the look and feel of the page itself, rather than the underlying technologies that would eventually be used implement it.

The mock-up that we eventually arrived at is shown in Figure 25. Since CAS 5 uses [Bootstrap][bootstrap-3] as its base HTML / CSS / JavaScript framework, we elected to develop our page using that framework as well, which gives us a responsive, mobile-first design. We augmented the base Bootstrap framework with Federico Zivolo's [Material Design for Bootstrap][md-for-bs-3] theme, which applies [Material Design][material-design] principles to Bootstrap's various elements. The New School's custom *Neue* font and CSS color settings bring the New School branding to the page.

{% include image.html file="ui/fig25-mock-up-login-page.png" alt="Browser Screen Shot" caption="Figure 25. Mock-up of the new login page" %}

The HTML for the mock-up page is actually pretty simple, and looks like this:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>

  <title>Log in - New School SSO</title>

  <!-- JQuery -->
  <script src="//code.jquery.com/jquery-1.10.2.min.js"></script>

  <!-- Bootstrap -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  <script src="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js">
    </script>

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

<body>
  <div class="container-fluid">
    <!-- New School header logo -->
    <header role="banner" class="tns-header">
      <section class="tns-header-region">
        <div class="tns-lockup">
          <div class="tns-banner text-center">
            <h1 class="tns-uname">
              <!-- Generator: Adobe Illustrator 18.1.0, SVG Export Plug-In.
                   SVG Version: 6.00 Build 0) --><svg xmlns="http://www.w3.org/2000/svg" xml:space="preserve"
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
  </div> <!-- container-fluid -->

  <!-- login form box -->
  <div id="container" class="container">
    <div id="content">
      <div class="row">
        <div class="col-sm-12 col-md-4 col-md-offset-4">
          <div class="well" id="login">
            <header class="tns-header">
              <div class="tns-banner">
                <h2 class="tns-sitename">Single Sign-On</h2>
              </div>
            </header>

            <h1>Log in</h1>
            <p>to continue to [name of service]</p>

            <form>
              <!-- inputs -->
              <div class="form-group label-floating is-empty">
                <label class="control-label" for="username">NetID</label>
                <input class="form-control" id="username" name="username"
                  type="text" style="cursor: auto;">
              </div>
              <div class="form-group label-floating is-empty">
                <label class="control-label" for="password">Password</label>
                <input class="form-control" id="password" name="password"
                  type="password" style="cursor: auto;">
              </div>

              <!-- button -->
              <div class="form-group text-center">
                <button class="btn btn-primary btn-lg" name="submit">
                  Continue
                  <div class="ripple-container"></div>
                </button>
              </div>

              <!-- account help -->
              <div class="form-group">
                <p class="acctopts">
                  <a href="https://account.newschool.edu/cgi-bin/acctservices.pl?f=i&s=pr">
                    <i class="material-icons acctopts">lock</i>
                    Reset your password
                  </a>
                </p>
                <p class="acctopts">
                  <a href="https://account.newschool.edu/cgi-bin/acctservices.pl?f=i&s=gn">
                    <i class="material-icons acctopts">help</i>
                    Look up your NetID
                  </a>
                </p>
              </div>

              <!-- privacy and terms -->
              <div class="form-row text-right privacy-terms">
                <a href="//www.newschool.edu/privacy-policy/" target="_blank">
                  <span>Privacy</span>
                </a>
                <a href="//it.newschool.edu/sites/default/files/uploads/documents/Statement%20on%20the%20Responsibilities%20of%20Computer%20Users%20v2.1.pdf" target="_blank">
                  <span>Terms</span>
                </a>
              </div> <!-- form-row -->
            </form>
          </div> <!-- well -->
        </div> <!-- col -->
      </div> <!-- row -->
    </div> <!-- content -->
  </div> <!-- container -->

  <script>
    $(function () {
      $.material.init();
   });
  </script>
</body>
</html>
```

The CSS styling for all the elements is included in the `newschool.css` file, and the *Neue* font is loaded by `tnsfonts.css`. The script at the bottom of the page initializes the Material Design for Bootstrap theme.

With the design finalized, the next step is to "port" the mock-up into the CAS user interface framework. This is described in the following pages.

## References

* [Bootstrap][bootstrap-3]
* [Material Design][material-design]
* [Material Design for Bootstrap][md-for-bs-3]

{% include reflinks.md %}
{% include links.html %}
