{% include "warning.html" %}

<h2 id="why">Why imports?</h2>

Think about how you load different types of resources on the web. For JS, we have `<script src>`. For CSS, your go-to is probably `<link rel="stylesheet">`. For images it's `<img>`. Video has `<video>`. Audio, `<audio>`.... Get to the point! The majority of the web's content has a simple and declarative way to load itself. Not so for HTML. Here's your options:

1. **`<iframe>`** - tried and true but heavy weight. An iframe's content lives entirely in a separate context than your page. While that's mostly a great feature, it creates additional challenges (shrink wrapping the size of the frame to its content is tough, insanely frustrating to script into/out of, nearly impossible to style).
-  **AJAX** - [I love `xhr.responseType="document"`](http://ericbidelman.tumblr.com/post/31140607367/mashups-using-cors-and-responsetype-document), but you're saying I need JS to load HTML? That doesn't seem right.
- **CrazyHacks&#8482;** - embedded in strings, hidden as comments (e.g. `<script type="text/html">`). Yuck!

See the irony? **The web's most basic content, HTML, requires the greatest amount
of effort to work with**. Fortunately, [Web Components](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html) are here to get us back on track.

<h2 id="started">Getting started</h2>

[HTML Imports](http://www.w3.org/TR/2013/WD-html-imports-20130514/), part of the [Web Components](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html) cast, is a way to include HTML documents in other HTML documents. You're not limited to markup either. An import can also include CSS, JavaScript, or anything else an `.html` file can contain. In other words, this makes imports a **fantastic tool for loading related HTML/CSS/JS**.

<h3 id="basics">The basics</h3>

Include an import on your page by declaring a `<link rel="import">`:

    <head>
      <link rel="import" href="/path/to/imports/stuff.html">
    </head>

The URL of an import is called an _import location_. To load content from another domain, the import location needs to be CORS-enabled:

    <!-- Resources on other origins must be CORS-enabled. -->
    <link rel="import" href="http://example.com/elements.html">

<p class="notice fact">The browser's network stack automatically de-dupes all requests from the same URL. This means that imports that reference the same URL are only retrieved once. No matter how many times an import at the same location is loaded, it only executes once. This means resources are automatically de-duped for you!</p>

<h3 id="featuredetect">Feature detection and support</h3>

To detect support, check if `.import` exists on the `<link>` element:

    function supportsImports() {
      return 'import' in document.createElement('link');
    }

    if (supportsImports()) {
      // Good to go!
    } else {
      // Use other libraries/require systems to load files.
    }

Browser support is still in the early days. Chrome 31 is the first browser to have an implementation. You can enable the flag by turning on **Enable HTML Imports** in `about:flags`. For other browsers, [Polymer's polyfill](http://www.polymer-project.org/platform/html-imports.html) works great until things are widely supported.

<figure>
  <img src="aboutflag.png">
  <figcaption><b>Enable HTML Imports</b> in <code>about:flags</code>.</figcpation>
</figure>

<p class="notice tip">Also <b>Enable experimental Web Platform features</b> to get the other bleeding edge web component goodies.</p>

<h3 id="bundling">Bundling resources</h3>

Imports provide convention for bundling HTML/CSS/JS (even other HTML Imports) into a single deliverable. It's an intrinsic feature, but a powerful one. If you're creating a theme, library, or just want to segment your app into logical chunks, giving users a single URL is compelling. Heck, you could even deliver an entire app via an import. Think about that for a second.

<blockquote class="commentary talkinghead">Using only one URL, you can package together a single relocatable bundle of web goodness for others to consume.
</blockquote>

A real-world example is [Bootstrap](http://getbootstrap.com/). Bootstrap is comprised of
individual files (bootstrap.css, bootstrap.js, fonts), requires JQuery for its plugins, and provides markup examples. Developers like à la carte flexibility. It allows them buy in to the parts of the framework _they_ want to use. That said, I'd wager your typical JoeDeveloper&#8482; goes the easy route and downloads all of Bootstrap.

Imports make a ton of sense for something like Bootstrap. I present to you, the future of loading Bootstrap:

    <head>
      <link rel="import" href="bootstrap.html">
    </head>

Users simply load an HTML Import link. They don't need to fuss with the scatter-shot
of files. Instead, the entirety of Bootstrap is managed and wrapped up in an import, bootstrap.html:

    <link rel="stylesheet" href="bootstrap.css">
    <link rel="stylesheet" href="fonts.css">
    <script src="jquery.js"></script>
    <script src="bootstrap.js"></script>
    <script src="bootstrap-tooltip.js"></script>
    <script src="bootstrap-dropdown.js"></script>
    ...

    <!-- scaffolding markup -->
    <template>
      ...
    </template>

Let this sit. It's exciting stuff.

<h3 id="events">Load/error events</h3>

The `<link>` element fires a `load` event when an import is loaded successfully
and `onerror` when the attempt fails (e.g. if the resource 404s). 

Imports try to load immediately. An easy way avoid headaches
is to use the `onload`/`onerror` attributes:

    <script async>
      function handleLoad(e) {
        console.log('Loaded import: ' + e.target.href);
      }
      function handleError(e) {
        console.log('Error loading import: ' + e.target.href);
      }
    </script>

    <link rel="import" href="file.html"
          onload="handleLoad(event)" onerror="handleError(event)">

<p class="notice tip">Notice the event handlers are defined before the import is loaded on the page. The browser tries to load the import as soon as it encounters the tag. If the functions don't exist yet, you'll get console errors for undefined function names.</p>

Or, if you're creating the import dynamically:

    var link = document.createElement('link');
    link.rel = 'import';
    link.href = 'file.html'
    link.onload = function(e) {...};
    link.onerror = function(e) {...};
    document.head.appendChild(link);

<h2 id="usingcontent">Using the content</h2>

Including an import on a page doesn't mean "plop the content of that file here". It means "parser, go off an fetch this document so I can use it". To actually use the content, you have to take action and write script.

A critical `aha!` moment is realizing that an import is just a document. In fact, the content of an import is called an *import document*. You're able to **manipulate the guts of an import using standard DOM APIs**!

<h3 id="importprop">link.import</h3>

To access the content of an import, use the link element's `.import` property:

    var content = document.querySelector('link[rel="import"]').import;

`link.import` is `null` under the following conditions:

- The browser doesn't support HTML Imports.
- The `<link>` doesn't have `rel="import"`.
- The `<link>` has not been added to the DOM.
- The `<link>` has been removed from the DOM.
- The resource is not CORS-enabled.

**Full example**

Let's say `warnings.html` contains:

    <div class="warning">
      <style scoped>
        h3 {
          color: red;
        }
      </style>
      <h3>Warning!</h3>
      <p>This page is under construction</p>
    </div>

    <div class="outdated">
      <h3>Heads up!</h3>
      <p>This content may be out of date</p>
    </div>

Importers can grab a specific portion of this document and clone it into their page:

    <head>
      <link rel="import" href="warnings.html">
    </head>
    <body>
      ...
      <script>
        var link = document.querySelector('link[rel="import"]');
        var content = link.import;

        // Grab DOM from warning.html's document.
        var el = content.querySelector('.warning');

        document.body.appendChild(el.cloneNode(true));
      </script>
    </body>

<div class="demoarea" id="warning-example-area"></div>

<link rel="import" id="warning-example-link" href="warning.html">
<script>
  var link = document.querySelector('#warning-example-link');
  if ('import' in link) {
    var content = link.import;
    var alertDOM = content.querySelector('div.alert');
    document.querySelector('#warning-example-area').appendChild(alertDOM.cloneNode(true));
  }
</script>

<h3 id="includejs">Scripting in imports</h3>

Imports are not in the main document. They're satellite to it. However, your import can still act on the main page even though the main document reigns supreme. An import can access its own DOM and/or the DOM of the page that's importing it:

**Example** - import.html that adds one of its stylesheets to the main page

    <link rel="stylesheet" href="http://www.example.com/styles.css">
    <link rel="stylesheet" href="http://www.example.com/styles2.css">
    ...

    <script>
      // importDoc references this import's document
      var importDoc = document.currentScript.ownerDocument;

      // mainDoc references the main document (the page that's importing us)
      var mainDoc = document;

      // Grab the first stylesheet from this import, clone it,
      // and append it to the importing document.
      var styles = importDoc.querySelector('link[rel="stylesheet"]');
      mainDoc.head.appendChild(styles.cloneNode(true));
    </script>

Notice what's going on here. The script inside the import references the imported document (`document.currentScript.ownerDocument`), and appends part of that document to the importing page (`mainDoc.head.appendChild(...)`). Pretty gnarly if you ask me.

<blockquote class="commentary talkinghead">A script in an import can either execute code directly, or define functions to be used by the importing page. This is similar to the way <a href="http://docs.python.org/2/tutorial/modules.html#more-on-modules">modules</a> are defined in Python.
</blockquote>

Rules of JavaScript in an import:

- Script in the import is executed in the context of the window that contains the importing `document`. So `window.document` refers to the main page document. This has two useful corollaries:
    - functions defined in an import end up on `window`.
    - you don't have to do anything crazy like append the import's `<script>` blocks to the main page. Again, script gets executed.
- Imports do not block parsing of the main page. However, scripts inside them are processed in order. This means you get defer-like behavior while maintaining proper script order. More on this below.

<h2 id="deliver-webcomponents">Delivering Web Components</h2>

The design of HTML Imports lends itself nicely to loading reusable content on the web. In particular, it's an ideal way to distribute Web Components. Everything from basic [HTML `<template>`](/webcomponents/template/)s to full blown [Custom Elements](/tutorials/webcomponents/customelements/#registering) with Shadow DOM [[1](/tutorials/webcomponents/shadowdom/), [2](/tutorials/webcomponents/shadowdom-201/), [3](/tutorials/webcomponents/shadowdom-301/)]. When these technologies are used in tandem, imports become a [`#include`](http://en.cppreference.com/w/cpp/preprocessor/include) for Web Components.

<h3 id="include-templates">Including templates</h3>

The [HTML Template](/tutorials/webcomponents/template/) element is a natural fit for HTML Imports. `<template>` is great for scaffolding out sections of markup for the importing app to use as it desires. Wrapping content in a `<template>` also gives you the added benefit of making the content inert until used. That is, scripts don't run until the template is added to the DOM). Boom!

import.html

    <template>
      <h1>Hello World!</h1>
      <img src="world.png"> <!-- !requested until the template goes live. -->
      <script>alert("Executed when the template is activated.");</script>
    </template>

index.html

    <head>
      <link rel="import" href="import.html">
    </head>
    <body>
      <div id="container"></div>
      <script>
        var link = document.querySelector('link[rel="import"]');

        // Clone the <template> in the import.
        var template = link.import.querySelector('template');
        var content = template.content.cloneNode(true)

        document.querySelector('#container').appendChild(content);
      </script>
    </body>

<h3 id="include-elements">Registering custom elements</h3>

[Custom Elements](tutorials/webcomponents/customelements/) is another Web Component technology that plays absurdly well with HTML Imports. [Imports can execute script](#includejs), so why not define + register your custom elements so users don't have to? Call it..."auto-registration". 

elements.html

    <script>
      // Define and register <say-hi>.
      var proto = Object.create(HTMLElement.prototype);
      
      proto.createdCallback = function() {
        this.innerHTML = 'Hello, <b>' +
                         (this.getAttribute('name') || '?') + '</b>';
      };

      document.register('say-hi', {prototype: proto});

      // Define and register <shadow-element> that uses Shadow DOM.
      var proto2 = Object.create(HTMLElement.prototype);

      proto2.createdCallback = function() {
        var root = this.createShadowRoot();
        root.innerHTML = "<style>::content > *{color: red}</style>" +
                         "I'm a " + this.localName +
                         " using Shadow DOM!<content></content>";
      };
      document.register('shadow-element', {prototype: proto2});
    </script>

This import defines (and registers) two elements, `<say-hi>` and `<shadow-element>`. The importer can simply declare them on their page. No wiring needed.

index.html

    <head>
      <link rel="import" href="elements.html">
    </head>
    <body>
      <say-hi name="Eric"></say-hi>
      <shadow-element>
        <div>( I'm in the light dom )</div>
      </shadow-element>
    </body>

<link rel="import" href="elements.html">

<div class="demoarea">
  <say-hi name="Eric"></say-hi><br><br>
</div>

<div class="demoarea">
  <shadow-element>
    <div>( I'm in the light dom )</div>
  </shadow-element>
</div>

In my opinion, this workflow alone makes HTML Imports an ideal way to share Web Components.

<h3 id="depssubimports">Managing dependencies and sub-imports</h3>

> Yo dawg. I hear you like imports, so I included an import _in_ your import.

<h4 id="sub-imports">Sub-imports</h4>

It can be useful for one import to include another. For example, if you want to reuse or extend another component, use an import to load the other element(s).

Below is a real example from [Polymer](http://polymer-project.org). It's a new tab component (`<polymer-ui-tabs>`) that reuses a layout and selector component. The dependencies are managed using HTML Imports. 

polymer-ui-tabs.html

    <link rel="import" href="polymer-selector.html">
    <link rel="import" href="polymer-flex-layout.html">

    <polymer-element name="polymer-ui-tabs" extends="polymer-selector" ...>
      <template>
        <link rel="stylesheet" href="polymer-ui-tabs.css">
        <polymer-flex-layout></polymer-flex-layout>
        <shadow></shadow>
      </template>
    </polymer-element>

[full source](https://github.com/Polymer/polymer-ui-elements/blob/master/polymer-ui-tabs/polymer-ui-tabs.html)

App developers can import this new element using:

    <link rel="import" href="polymer-ui-tabs.html">
    <polymer-ui-tabs></polymer-ui-tabs>

When a new, more awesome `<polymer-selector2>` comes along in the future, you can swap out `<polymer-selector>` and start using it straight away. You won't break your users thanks to imports and web components.

<h4 id="deps">Dependency management</h4>

We all know that loading JQuery more than once per page causes errors. Isn't this
going to be a _huge_ problem for Web Components when multiple components use the same library? Not if we use HTML Imports! They can be used to manage dependencies.

By wrapping libraries in an HTML Import, you automatically de-dupe resources.
The document is only parsed once. Scripts are only executed once. Say you define an import, jquery.html, that includes jquery.js:

jquery.html

    <script src="jquery.js"></script>

In subsequent imports, that import gets reused:

ajax-element.html

    <link rel="import" href="jquery.html">

    <script>
      var proto = Object.create(HTMLElement.prototype);

      proto.makeRequest = function(url, done) {
        return $.ajax(url).done(function() { ... });
      };

      document.register('ajax-element', {prototype: proto});
    </script>

index.html (main page)

    <head>
      <link rel="import" href="jquery.html">
      <link rel="import" href="ajax-element.html">
    </head>
    <body>

    ...

    <script>
      $(document).ready(function() {
        var el = document.createElement('ajax-element');
        el.makeRequest('http://example.com');
      });
    </script>
    </body>

Despite JQuery being used in many places, it's only loaded once because the browser is reusing the import.

<h2 id="performance">Performance considerations</h2>

HTML Imports are totally awesome but as with any new web technology, you should
use them wisely. Web development best practices still hold true. Below are some things to keep in mind.

<h3 id="perf-concat">Concatenate imports</h3>

Reducing network requests is important. [Vulcanizer](https://github.com/Polymer/vulcanize) is an npm build tool from the [Polymer](http://www.polymer-project.org/) team that recursively flattens a set of HTML Imports into a single file. Think of it as a concatenation build step for Web Components.

<h3 id="perf-caching">Imports leverage browser caching</h3>

Many people often forget that the browser's networking stack has been finely tuned over the years. Imports (and sub-imports) take advantage of this logic too.

<h3 id="perf-inert">Content is useful only when you add it</h3>

Think of content as inert until you call upon its services. Take a normal, dynamically created stylesheet:

    var link = document.createElement('link');
    link.rel = 'stylesheet';
    link.href = 'styles.css';

The browser won't request styles.css until `link` is added to the DOM:

    document.head.appendChild(link); // browser requests styles.css

Another example is dynamically created markup:

    var h2 = document.createElement('h2');
    h2.textContent = 'Booyah!';

The `h2` is relatively meaningless until you add it to the DOM.

The same concept holds true for the import document. Unless you append it's content to the DOM, it's a no-op. In fact, the only thing that "executes" in the import document directly is `<script>`. See [scripting in imports](#includejs).

<h3 id="perf-parsing">Optimizing for async loading</h3>

**Imports don't block parsing of the main page**. Scripts inside imports are processed in order but don't block the importing page. This means you get defer-like behavior while maintaining proper script order. One benefit of putting your imports in the `<head>` is that it lets the parser start working on the content as soon as possible. With that said, it's critical to remember `<script>` in the main document *still* continues to block the page:

    <head>
      <link rel="import" href="/path/to/import_that_takes_5secs.html">
      <script>console.log('I block page rendering');</script>
    </head>

Depending on your app structure and use case, there are several ways to optimize async behavior. The techniques below mitigate blocking the main page rendering.

**Scenario #1 (preferred): you don't have script in `<head>` or inlined in `<body>`**

My recommendation for placing `<script>` is to avoid immediately following your imports. Move scripts as late in the game as possible...but you're already doing that best practice, AREN'T YOU!? ;)

Here's an example:

    <head>
      <link rel="import" href="/path/to/import.html">
      <link rel="import" href="/path/to/import2.html">
      <!-- avoid including script -->
    </head>
    <body>
      <!-- avoid including script -->

      <div id="container"></div>

      <!-- avoid including script -->
      ...

      <script>
        // Other scripts n' stuff.

        // Bring in the import content.
        var link = document.querySelector('link[rel="import"]');
        var post = link.import.querySelector('#blog-post');
        
        var container = document.querySelector('#container');
        container.appendChild(post.cloneNode(true));
      </script>
    </body>

Everything is at the bottom.

**Scenario 1.5: the import adds itself**

Another option is to have the import [add its own content](#includejs). If the
import author establishes a contract for the app developer to follow, the import can add itself to an area of the main page:

import.html:

    <div id="blog-post">...</div>
    <script>
      var me = document.currentScript.ownerDocument;
      var post = me.querySelector('#blog-post');

      var container = document.querySelector('#container');
      container.appendChild(post.cloneNode(true));
    </script>

index.html

    <head>
      <link rel="import" href="/path/to/import.html">
    </head>
    <body>
      <!-- no need for script. the import takes care of things -->
    </body>

**Scenario #2: you *have* script in `<head>` or inlined in `<body>`**

If you have an import that takes a long time to load, the first `<script>` that follows it on the page will block the page from rendering. Google Analytics for example, 
recommends putting the tracking code in the `<head>`, If you can't avoid putting `<script>` in the `<head>`, dynamically adding the import will prevent blocking the page:

    <head>
      <script>
        function addImportLink(url) {
          var link = document.createElement('link');
          link.rel = 'import';
          link.href = url;
          link.onload = function(e) {
            var post = this.import.querySelector('#blog-post');

            var container = document.querySelector('#container');
            container.appendChild(post.cloneNode(true));
          };
          document.head.appendChild(link);
        }

        addImportLink('/path/to/import.html'); // Import is added early :)
      </script>
      <script>
        // other scripts
      </script>
    </head>
    <body>
       <div id="container"></div>
       ...
    </body>

Alternatively, add the import near the end of the `<body>`:

    <head>
      <script>
        // other scripts
      </script>
    </head>
    <body>
      <div id="container"></div>
      ...

      <script>
        function addImportLink(url) { ... }

        addImportLink('/path/to/import.html'); // Import is added very late :(
      </script>
    </body>

<p class="notice"><b>Note:</b> This very last approach is least preferable. The parser doesn't start to work on the import content until late in the page.</p>

<h2 id="tips">Things to remember</h2>

- An import's mimetype is `text/html`.

- Resources from other origins need to be CORS-enabled.

-  Consecutive imports from the same URL are retrieved once. That shouldn't be surprising given that stylseheets behave the same way. For example:

        <link rel="import" href="file.html">
        <link rel="import" href="file.html">

    results in a single request for `file.html`.

- Scripts in an import are processed in order, but do not block the main document parsing.

- An import link doesn't mean "#include the content here". It means "parser, go off an fetch this document so I can use it later". While scripts execute at import time, stylesheets, markup, and other resources need to be added to the main page explicitly This is a major difference between HTML Imports and `<iframe>`, which says "load and render this content here".

<h2 id="conclusion">Conclusion</h2>

HTML Imports allow bundling HTML/CSS/JS as a single resource. While useful by themselves, this idea becomes extremely powerful in the world of Web Components. Developers can create reusable components for others to consume and bring in to their own app, all delivered through `<link rel="import">`.

HTML Imports are a simple concept, but enable a number of interesting use cases
for the platform.

<h3 id="usecases">Use cases</h3>

- **Distribute** related [HTML/CSS/JS as a single bundle](#bundling). Theoretically, you could import an entire web app into another.
- **Code organization** - segment concepts logically into different files, encouraging modularity &amp; reusability**.
- **Deliver** one or more [Custom Element](/tutorials/webcomponents/customelements/) definitions. An import can be used to [register](/tutorials/webcomponents/customelements/#registering) and include them in an app. This practices good software patterns, keeping the element's interface/definition separate from how its used.
- [**Manage dependencies**](#depssubimports) - resources are automatically de-duped.
- **Chunk scripts** - before imports, a large-sized JS library would have its file wholly parsed in order to start running, which was slow. With imports, the library can start working as soon as chunk A is parsed. Less latency!

      `<link rel="import" href="chunks.html">`

        <script>/* script chunk A goes here */</script>
        <script>/* script chunk B goes here */</script>
        <script>/* script chunk C goes here */</script>
        ...

- **Parallelizes HTML parsing** - first time the browser has been able to run two (or more) HTML parsers in parallel.

- **Enables switching between debug and non-debug modes** in an app, just by changing the import target itself. Your app doesn't need to know if the import target is a bundled/compiled resource or an import tree. 