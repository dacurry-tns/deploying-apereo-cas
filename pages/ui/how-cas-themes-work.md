---
title: How CAS themes work
last_updated: October 10, 2018
sidebar: main_sidebar
permalink: ui_how-cas-themes-work.html
summary:
---

CAS uses Spring Web Flow to perform its processing of login and logout requests (as well as "add-ons" like multi-factor authentication). It's not necessary to understand the intricacies of Spring Web Flow to customize the CAS user interface, but it does help to have a basic understanding of what's going on behind the scenes.

A *flow* encapsulates a reusable sequence of steps that guide a user through the completion of some task, such as logging in or logging out (the two main flows provided by CAS). Flows can span multiple HTTP requests, have state, deal with transactional data, are reusable, and may be dynamic and long-running. Each step in a flow is called a *state*. There are five kinds of state:

1. A *view state* uses a dynamically-generated web page, or *view*, to display information to the user or prompt the user for input. Transition to another state is triggered by an event on the web page, such as the user clicking a "submit" button.
2. An *action state* calls a CAS server function and then transitions to another state in the flow depending on the outcome of the function call.
3. A *decision state* evaluates a Boolean expression and then transitions to one of two other states in the flow depending on whether the expression is true or false.
4. A *subflow state* allows one flow to call another flow to perform some steps. Data may be passed between the calling flow and the subflow.
5. An *end state* defines the end of a flow. If the end state is part of the root flow, execution ends. If it is part of a subflow, the calling flow is resumed.

The CAS user interface&mdash;the login page, logout page, and other pages displayed for multi-factor authentication and so on&mdash;is part of the view state. The other states comprise the "business logic" that determines which page(s) (view state(s)) are ultimately shown to the user.

## Parts of the user interface

There are three "parts" to the CAS user interface:

1. **Views.** A set of HTML files, one per view state, as described above. View pages are processed through the Thymeleaf template engine enabling them to dynamically access property settings and variables and evaluate logical expressions using their values. Thymeleaf also makes it possible to define a common layout template (page background, header and footer, navigation elements, style sheet inclusions, etc.) to be shared by all the view pages.
2. **Themes.** A collection of cascading style sheets, JavaScript files, and image files that are included by the view pages. The CSS files control the colors, fonts, and other style-related aspects of the interface. The JavaScript files define procedures to be executed in the user's browser to handle things like caps-lock detection, multi-factor authentication, etc. The image files include logos, buttons, lines, etc. used by the style sheets.
3. **Message bundles.** Java property files (`messages.properties` for English, `messages_xx.properties` for locale `xx`) that define all the text messages displayed in the various views. The Thymeleaf template engine automatically inserts messages from the proper message bundle (based on the locale setting) into the HTML views through property reference substitution.

The default views, default theme, and default message bundles are automatically included in the CAS WAR file by the Maven build process. The message bundles reside under `WEB-INF/classes`, the default theme resides under `WEB-INF/classes/static` (in `css`, `js`, and `images` subdirectories), and the default views reside under `WEB-INF/classes/templates`.

The default theme itself is defined by the `WEB-INF/classes/cas-default-theme.properties` file:

```properties
standard.custom.css.file=/css/cas.css
admin.custom.css.file=/css/admin.css
cas.javascript.file=/js/cas.js
```

These properties define the main CSS file for the user views, the main CSS file for the dashboard (admin pages), and the main JavaScript file for the user views, respectively. The paths are rooted at `WEB-INF/classes/static` in the WAR file, which serves as the document root for the CAS web application. This means, for example, that the complete URL for `/css/cas.css` is `https://casdev.newschool.edu/cas/css/cas.css`, which, if accessed, will retrieve the file `/var/lib/tomcat/webapps/cas/WEB-INF/classes/static/css/cas.css`.

## Modifying the user interface

When modifying the CAS user interface, there are two options: change the decorative elements (styles and scripts) of the user interface but keep the same structural elements (HTML views), or change both the decorative elements and the structural elements.

### Changing decorative elements (styles and scripts)

To define a new theme that will re-style the default views, a `WEB-INF/classes/[theme_name].properties` file that specifies the location of the main CSS and JavaScript files for the theme should be created:

```properties
standard.custom.css.file:   /themes/[theme_name]/css/cas.css
cas.javascript.file:        /themes/[theme_name]/js/cas.js
admin.custom.css.file:      /themes/[theme-name]/css/admin.css
```

Next, a `WEB-INF/classes/static/themes/[theme_name]` directory should be created, and theme-specific `cas.css`, `admin.css`, and `cas.js` files placed in the appropriate subdirectories (`css` and `js`). Additional subdirectories for `images` and `fonts` can be created here too, if needed.

{% include tip.html content="When you define a custom theme, the CSS and JavaScript files you declare in the `[theme_name].properties` file will be included *instead of* the CSS and JavaScript files from the default theme. If your intention is to keep most of the styling and scripting the same and make only make decorative changes to the default theme (e.g., changing the background color or font size), you may wish to begin with copies of the default files (`WEB-INF/classes/static/css/cas.css` and `WEB-INF/classes/static/js/cas.js`) and add your customizations to them rather than starting with empty files." %}

### Changing structural elements (HTML views)

By default, the new theme created above will be applied to the default views. However, it's also possible to define a new set of views, different from the defaults. To do this, the `[theme_name].properties` file and the locations for the decorative components should be created as described above, and then a `WEB-INF/classes/templates/[theme_name]` directory should be created to hold the HTML view files for the new theme. There is no property setting to inform the server that it should use the new set of views; it will simply use the files in the `WEB-INF/classes/templates/[theme_name]` directory if it exists, or the default files (in `WEB-INF/classes/templates`) if it does not.

{% include note.html content="If the `WEB-INF/classes/templates/[theme_name]` directory exists, the server will expect to find *all* the views in that directory. Even if you're only planning to change one or two of the views (e.g., the login and logout pages), you must provide an HTML file for every view that the configured web flows might need. The easiest way to accomplish this is to copy the entire contents of the default template directory (all files and subdirectories) into your custom template directory and then modify the files for the views you wish to customize, rather than starting with an empty directory." %}

### Changing message strings

By default, Thymeleaf will automatically retrieve message strings (using property names) from `WEB-INF/classes/messages.properties` if the English (`en`) locale is in effect, or from `WEB-INF/classes/messages_xx.properties` if another (`xx`) locale is in effect. To change the value of a particular message string, the property that defines that string can be re-defined in `WEB-INF/classes/custom_messages.properties`. New properties can also be defined in this file and used in the HTML views.

## Summary

The table below summarizes the various files and directories associated with customizing the CAS user interface. The procedures for actually creating them and including them in the CAS WAR file are described in the following sections.

<table width="100%">
    <colgroup>
        <col width="24%" />
        <col width="38%" />
        <col width="38%" />
    </colgroup>
    <thead>
        <tr>
            <th markdown="span"></th>
            <th markdown="span">CAS default theme</th>
            <th markdown="span">Custom theme</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td markdown="span">Theme definition</td>
            <td markdown="span">`WEB-INF/classes/`<br>&nbsp;&nbsp;&nbsp;`cas-default-theme.properties`</td>
            <td markdown="span">`WEB-INF/classes/`<br>&nbsp;&nbsp;&nbsp;`[theme_name].properties`</td>
        </tr>
        <tr>
            <td markdown="span">Style sheets</td>
            <td markdown="span">`WEB-INF/classes/static/`<br>&nbsp;&nbsp;&nbsp;`css/*`</td>
            <td markdown="span">`WEB-INF/classes/static/`<br>&nbsp;&nbsp;&nbsp;`themes/[theme_name]/css/*`</td>
        </tr>
        <tr>
            <td markdown="span">JavaScript</td>
            <td markdown="span">`WEB-INF/classes/static/`<br>&nbsp;&nbsp;&nbsp;`js/*`</td>
            <td markdown="span">`WEB-INF/classes/static/`<br>&nbsp;&nbsp;&nbsp;`themes/[theme_name]/js/*`</td>
        </tr>
        <tr>
            <td markdown="span">Images</td>
            <td markdown="span">`WEB-INF/classes/static/`<br>&nbsp;&nbsp;&nbsp;`images/*`</td>
            <td markdown="span">`WEB-INF/classes/static/`<br>&nbsp;&nbsp;&nbsp;`themes/[theme_name]/images/*`</td>
        </tr>
        <tr>
            <td markdown="span">HTML Views<br>&nbsp;&nbsp;&nbsp;</td>
            <td markdown="span">`WEB-INF/classes/`<br>&nbsp;&nbsp;&nbsp;`templates/*`</td>
            <td markdown="span">`WEB-INF/classes/`<br>&nbsp;&nbsp;&nbsp;`templates/[theme_name]/*`</td>
        </tr>
        <tr>
            <td markdown="span">Messages<br>&nbsp;&nbsp;&nbsp;</td>
            <td markdown="span">`WEB-INF/classes/`<br>&nbsp;&nbsp;&nbsp;`messages.properties`<br>&nbsp;&nbsp;&nbsp;`messages_xx.properties`</td>
            <td markdown="span">`WEB-INF/classes/`<br>&nbsp;&nbsp;&nbsp;`custom_messages.properties`</td>
        </tr>
    </tbody>
</table>

## References

* [CAS 5: User Interface Customization: Dynamic Themes][casdoc-ui-themes]
* [CAS 5: User Interface Customization: CSS/JavaScript][casdoc-ui-cssjs]
* [CAS 5: User Interface Customization: Views][casdoc-ui-views]
* [CAS Community: "CAS 5.1.x Custom template. Anyone get this working?"][cas-user-ng]
* [CAS Community: "CAS 5.2.0-RC1 Theme not resolved correctly"][cas-user-chase]

{% include reflinks.md %}
{% include links.html %}
