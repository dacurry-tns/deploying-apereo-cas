---
title: How Thymeleaf layouts work
last_updated: October 10, 2018
sidebar: main_sidebar
permalink: ui_how-thymeleaf-layouts-work.html
summary:
---

As described in the [previous section][ui_how-cas-themes-work], CAS displays a dynamically-created web page, or *view*, whenever the web flow enters a view state. CAS 5 uses Thymeleaf, a server-side template engine, to manage the generation of this dynamic content (CAS 3 used Java Server Pages for this purpose).

Thymeleaf uses *processors* to make dynamic changes to a document. A processor may perform variable (or property) substitutions, evaluate expressions, implement conditional statements, or pass information to/from application methods. In HTML documents, processors take the form of specially-defined HTML attributes that are evaluated by the Thymeleaf engine. A set of processors&mdash;plus some extra artifacts&mdash; is called a *dialect*. Thymeleaf comes with two dialects out of the box: the *Standard Dialect*, which defines a base set of features that should be more than enough for most scenarios, and the *SpringStandard Dialect*, which extends the Standard Dialect with some specific features that integrate Thymeleaf with Spring MVC applications.

CAS makes use of the SpringStandard Dialect and another dialect, called the *Thymeleaf Layout Dialect*.

## Thymeleaf Layout Dialect

The Thymeleaf Layout Dialect is a dialect (feature set) for Thymeleaf that allows layouts and reusable templates to be built in order to improve code reuse. The dialect makes use of two major components: layouts and content templates.

### Layouts

A *layout* is an HTML file that defines web page components that will be common to all web pages using the layout: the header, footer, menu, navigation, etc. A very simple layout might look like this:

```html
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
<head>
  <title>My simple layout</title>
  <link rel="stylesheet" href="common-styles.css"/>
  <script src="common-script.js"></script>
</head>
<body>
  <header>
    <h1>My Little Website</h1>
  </header>
  <section layout:fragment="content">
    <p>Default page content</p>
  </section>
  <footer>
    <p>My little footer</p>
    <p layout:fragment="custom-footer">Default footer text</p>
  </footer>  
</body>
</html>
```

The interesting parts of this layout are the `layout:fragment` attribute used on the `<section>` element (line 12) and the `<p>` element in the footer (line 17). These elements are candidates to be replaced by matching fragments from content templates.

### Content templates

Content templates provide content to be substituted into a layout. A simple content template might look something like this:

```html
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{layout}">
<head>
  <title>Some simple content</title>
  <link rel="stylesheet" href="content-specific-styles.css"/>
  <script src="content-specific-script.js"></script>
</head>
<body>
  <p>Here is some text from the content template.</p>
  <section layout:fragment="content">
    <p>This is a paragraph from the content template.</p>
  </section>
  <footer>
    <p layout:fragment="custom-footer">This is the custom footer from the content template.</p>
  </footer>
</body>
</html>
```

The `layout:decorate` attribute on the `<html>` tag is required, and tells Thymeleaf which layout will be "decorated" using this content template. The value in the curly braces is the name of the layout; Thymeleaf will look for a file with that name and a `.html` suffix to find the file containing the layout. The content template above defines its own title, style sheet, and script file; it also defines *fragments* named `content` and `custom-footer`.

After Thymeleaf processes the content template, the resulting page sent to the user's browser will be:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Some simple content</title>
  <link rel="stylesheet" href="common-styles.css"/>
  <script src="common-script.js"></script>
  <link rel="stylesheet" href="content-specific-styles.css"/>
  <script src="content-specific-script.js"></script>
</head>
<body>
  <header>
    <h1>My Little Website</h1>
  </header>
  <section>
    <p>This is a paragraph from the content template.</p>
  </section>
  <footer>
    <p>My little footer</p>
    <p>This is the custom footer from the content template.</p>
  </footer>
</body>
</html>
```

The content template decorated `layout.html`, resulting in a combination of the layout plus the fragments of the content template:
* The body of the content template's `<head>` element has been appended to the body of the layout's `<head>` element to produce a single, merged `<head>` element.
* The body of the content template's `<title>` element has replaced (overridden) the body of the layout's `<title>` element.
* The elements in the layout with a `layout:fragment` attribute (the `<section>` and `<p>` elements mentioned earlier) have been replaced with the contents of the corresponding fragments from the content template.
* Anything *outside* of a `layout:fragment` in the `<body>` of the content template (e.g., "Here is some text from the content template") has been discarded.
* All other elements in the layout have been reproduced verbatim.

Content templates don't have to provide definitions for all fragments referenced by the layout template. For example:

```html
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{layout}">
<head>
<body>
  <section layout:fragment="content">
    <p>This is a paragraph from the content template.</p>
  </section>
</body>
</html>
```

would result in:

```html
<!DOCTYPE html>
<html>
<head>
  <title>My simple layout</title>
  <link rel="stylesheet" href="common-styles.css"/>
  <script src="common-script.js"></script>
</head>
<body>
  <header>
    <h1>My Little Website</h1>
  </header>
  <section>
    <p>This is a paragraph from the content template.</p>
  </section>
  <footer>
    <p>My little footer</p>
    <p>Default footer text</p>
  </footer>  
</body>
</html>
```

In this case, the page title is the one defined in the layout, because the content template did not define one. And the text from the layout also appears in the page footer, because the content template did not define a `custom-footer` fragment.

### The `layout:title-pattern` attribute

As shown above, if the content template defines a `<title>` element, the content of that element will replace the content of a `<title>` element defined in the layout template. But for occasions in which the page title includes both a "standard" part and a "variable" part, Thymeleaf provides the `layout:title-pattern` attribute for use in the layout template. A layout that contains

```html
<title layout:title-pattern="$LAYOUT_TITLE - $CONTENT-TITLE">Roses are red</title>
```

and a content template that contains

```html
<title>Violets are blue</title>
```

would result in

```html
<title>Roses are red - Violets are blue</title>
```

Alternatively, a layout contains

```html
<title layout:title-pattern="$CONTENT_TITLE - $LAYOUT-TITLE">Roses are red</title>
```

and the same content template would result in

```html
<title>Violets are blue - Roses are red</title>
```

## Thymeleaf SpringStandard Dialect

As shown above the Thymeleaf Layout Dialect defines some special HTML attributes (`layout:decorate`, `layout:fragment`, and `layout:title-pattern`, among others) that allow templates to be processed and turned into web pages. The rest of the Thymeleaf functionality used by CAS is provided by the SpringStandard Dialect (which, as mentioned above, is an extension of the Standard Dialect). The dialect features most commonly used in CAS views are described in the following subsections; for a complete description of all features, consult the documentation linked at the bottom of this page.

### Value substitutions

Thymeleaf allows values to be retrieved from Java properties, methods, classes, variables, etc.  

`#themes.code('property.name')`
:    Retrieve the value of `property.name` from the `[theme_name].properties` file and substitute it here.

`#{message.key}`
:    Look up the value of `message.key` in the `messages.properties` (or `messages_xx.properties`) bundle, or the overriding `custom_messages.properties` bundle, and substitute it here.

`${expression}`
:    Evaluate the expression, written in Spring Expression Language, and substitute the result here. The expression may contain, among other things, literals, boolean and relational operators, regular expressions, class expressions, method invocations, variables, bean references, access to properties, etc. Usually, in CAS templates, the expression is either a simple variable reference or method call to obtain a value.

`@{url}`
:    Treat `url` (which may be an expression) as a URL. This construct is usually used in conjunction with one of the attribute modifiers below.

### Attribute modifiers

Thymeleaf allows the values for attributes of HTML elements to be computed dynamically at runtime and "injected" into the element. To do this, special Thymeleaf variants of the attributes are used in place of the regular attributes. For example:

```html
<a th:href="@{#{screen.welcome.privacy.url}}">Privacy Policy</a>
```

will look up the value of `screen.welcome.privacy.url` in `messages.properties` and then set the `href` attribute of the `<a>` tag to this URL.

There are several dozen attributes for which Thymeleaf will perform this service, but the most commonly used ones in the CAS views are `th:href` and `th:src` for URL attributes (mostly used on `<a>`, `<img>`, `<script>`, and `<link>` tags), and `th:value` for HTML form input element default values.

### Text (tag body) modifiers

Thymeleaf also allows the values (contents) of HTML elements themselves to be computed dynamically at runtime. For example:

```html
<p th:text="#{home.welcome}">Welcome to our widget store!</p>
```

will look up the value of `home.welcome` in `messages.properties` and then set the contents of the `<p>` element to that value, replacing the words "Welcome to our widget store!".

The `th:text` modifier substitutes "escaped" text into an element, while the `th:utext` modifier substitutes "unescaped" text. The difference can be seen when the string to be substituted contains HTML tags or entity references instead of simple plain text. For example, if `messages.properties` contains

```properties
home.welcome=Welcome to our <b>fantastic</b> widget store!
```

then the example above will result in

```html
<p>Welcome to our &lt;b&gt;fantastic&lt;/b&gt; widget store!</p>
```

which is probably not what was intended. On the other hand, this:

```html
<p th:utext="#{home.welcome}">Welcome to our widget store!</p>
```

will produce

```html
<p>Welcome to our <b>fantastic</b> widget store!</p>
```

which is the intended result.

### Conditional evaluation

The `th:if` and `th:unless` attributes will evaluate an expression and, depending on the result, cause the element containing the attribute to be included in the output (what's sent to the browser) or not. The `th:if` attribute will cause the element to be included if the expression evaluates to true; the `th:unless` attribute will cause the element to be included if the expression evaluates to false. For example, the CAS login form contains this:

```html
<div class="registered-service" th:if="${registeredService}">
  <div th:if="${serviceUIMetadata}">
    <p>to continue to
       <span th:text="${serviceUIMetadata.displayName}"></span></p>
  </div>
  <div th:unless="${serviceUIMetadata}">
    <p>to continue to
      <span th:text="${registeredService.name}"></span></p>
  </div>
</div>
```

The outermost `<div>` and its contents will be included in the output only if the `registeredService` variable is set and non-null. If the variable is set, then the string `to continue to [service_name]` will be displayed. Depending on whether or not the `serviceUIMetadata` variable is set and non-null (i.e., whether the service is a SAML-based service or not), the value of `[service_name]` will be obtained from the SAML2 metadata or the service registry entry.

### Re-usable fragments

Thymeleaf also makes it possible to include fragments from other files, making it possible to define a fragment that can be used in more than one content template. For example, the line

```html
<div th:replace="fragments/footer"/>
```

will replace the `<div>` element (the "host tag") with the contents of the file `fragments/footer.html`. The `th:replace` directive may be used in both layout templates and content templates; it may also be used within the fragments themselves (e.g., `content.html` may contain `th:reaplce="fragments/foo"`, which in turn may contain `th:replace="fragments/bar"`). There is also a `th:insert` directive that will insert the contents of the fragment into the host tag, rather than replacing the host tag.

## The CAS templates

The CAS server's default Thymeleaf templates are kept in `WEB-INF/classes/templates`:

```console
casdev-master# cd /var/lib/tomcat/cas/WEB-INF/classes/templates
casdev-master# ls -F
casAcceptableUsagePolicyView.html
casAccountDisabledView.html
casAccountLockedView.html
...
casLoginView.html
casLogoutView.html
...
fragments/
layout.html
...
casdev-master#  
```

There are a little more than three dozen files whose names begin with `cas`; these are the content templates for the various views. There is one file for each view state defined by the CAS web flows and subflows; the names of the files make it fairly obvious which file belongs to each state.

The `layout.html` file is the layout template for all the views; this is where the main components of the user interface look and feel are defined.

The `fragments` directory contains re-usable HTML code fragments that are included into other templates with `th:replace`.

## References

* [Thymeleaf Layout Dialect: Examples][thymeleaf-layout-examples]
* [Thymeleaf: Tutorial: Using Thymeleaf][thymeleaf-using-tut]
* [Thymelead: Tutorial: Thymeleaf + Spring][thymeleaf-spring-tut]

{% include reflinks.md %}
{% include links.html %}
