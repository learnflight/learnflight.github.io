---
layout: post
title: "How to create a menu with Flight.js"
date: 2013-09-06 15:56
comments: true
categories:
---

We are going to create a simple navigation menu with
[Flight.js](http://flightjs.github.io/). The purpose of
this post is to illustrate a practical Flight.js example. Here are the
[docs](https://github.com/flightjs/flight/tree/master/doc) and the [github
page](https://github.com/flightjs/flight)
if you want to check them out.

{% img /images/navigation-menu.png %}

[View source](https://github.com/rogeliog/learn-flight-navigation-menu-demo/tree/gh-pages) or [Live demo](http://rogeliog.github.io/learn-flight-navigation-menu-demo).

# Initial Setup

We need to setup some boring stuff to be sure that we all have the same beginning
state. The fun stuff starts in the next section.

Before we start be sure to have installed [npm]() and [bower]().

Lets create a new folder for our project. Inside our folder lets setup our project.
We need to install Flight and requirejs to load the modules.

```bash
bower init
bower install flight --save
bower install requirejs --save
```

Once installed it we should a `bower.json` file and a `bower_components` dir

### Add the markup

We are going to create an `index.html` file in our directory that will have a
simple page.

{% codeblock index.html lang:html %}
<html>
  <head>
    <link rel="stylesheet" href="css/main.css">
  </head>
  <body>
    <div id="main">
      <nav id="menu">
        <a href="#" class="home menu-item">Home</a>
        <a href="#" class="projects menu-item">Projects</a>
        <a href="#" class="services menu-item">Services</a>
        <a href="#" class="contact menu-item">Contact</a>
      </nav>

      <div class="menu-content">
        Please click a menu item
      </div>
    </div>
    <script src="bower_components/jquery/jquery.js"></script>
    <script src="bower_components/requirejs/require.js" data-main="js/main.js"></script>
  </body>
</html>
{% endcodeblock %}


### Paste some CSS

We also need to paste some CSS to make it a look a bit sexier.

{% codeblock css/main.css lang:css %}
body {
  font: 15px/1.3 'Open Sans', sans-serif;
  color: #5e5b64;
  text-align:center;
}

#menu {
  display: inline-block;
  margin: 60px auto 45px;
  background-color: #15a5ec;
  box-shadow: 0 1px 1px #ccc;
}

#menu .menu-item {
  display: inline-block;
  padding: 20px 30px;
  color: #fff;
  font-weight: bold;
  font-size: 16px;
  text-decoration: none;
  text-transform: uppercase;
  -webkit-transition:background-color 0.20s;
  -moz-transition:background-color 0.20s;
  transition:background-color 0.20s;
}

#menu.home .home,
#menu.projects .projects,
#menu.services .services,
#menu.contact .contact{
  background-color: #999;
}


.menu-content {
  font-size:18px;
  color:#fff;
  padding: 30px 10px;
  background-color:#c4d7e0;
  text-transform: uppercase;
}
{% endcodeblock %}

# Lets start

We have one last "setup" duty to make. The `index.html` file has this line

{% codeblock index.html lang:html %}
<script src="bower_components/requirejs/require.js" data-main="js/main.js"></script>
{% endcodeblock %}

This line will reference and include our `js/main.js` file. In this file we need
to tell require.js that we are using the flight module and load the default
page.

{% codeblock js/main.js lang:js %}
'use strict';

requirejs.config({
  baseUrl: '',
  paths: {
    'flight': 'bower_components/flight',
    'component': 'js/component',
    'page': 'js/page'
  }
});

require(
  [
    'flight/lib/compose',
    'flight/lib/registry',
    'flight/lib/advice',
    'flight/lib/logger',
    'flight/lib/debug'
  ],

  function(compose, registry, advice, withLogging, debug) {
    debug.enable(true);
    compose.mixin(registry, [advice.withAdvice, withLogging]);

    require(['page/default'], function(initializeDefault) {
      initializeDefault();
    });
  }
);
{% endcodeblock %}

## Create the components

Now we can start creating our components. In this case we are going to create
three of them, one for the menu items, one for the menu content and one that
will wrap the menu.

**menu_item.js** Listens to clicks on the menu items and triggers a
  `uiMenuContentRefreshRequested` event to notify that we need to change the
  content.

{% codeblock js/component/menu_item.js lang:js %}
define(function (require) {
  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(menuItem);

  function menuItem() {
    this.select = function (e) {
      var value = this.$node.text();
      this.trigger('uiMenuContentRefreshRequested', {
        section: value,
        markup: '<b>' + value + '</b>'
      });
    }

    this.after('initialize', function () {
      this.on('click', this.select);
    });
  }
});
{% endcodeblock %}

**menu.js** Listens to the `uiMenuContentRefreshServed` event and adds the
  apropiate class to the menu.

{% codeblock js/component/menu.js lang:js %}
define(function (require) {
  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(menu);

  function menu() {
    this.setSelectedClass = function (e, data) {
      this.$node.removeClass().addClass(data.section);
    }

    this.after('initialize', function () {
      this.on(document, 'uiMenuContentRefreshServed', this.setSelectedClass);
    });
  }
});
{% endcodeblock %}

**menu_content.js** Listens to `uiMenuContentRefreshRequested`, updates the content
  with the appropiate markup and triggers a `uiMenuContentRefreshServed` to
  notify that content changed.

{% codeblock js/component/menu_content.js lang:js %}
define(function (require) {
  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(menuContent);

  function menuContent() {
    this.setText = function (e, data) {
      this.$node.html(data.markup);
      this.trigger('uiMenuContentRefreshServed', data);
    }

    this.after('initialize', function () {
      this.on(document, 'uiMenuContentRefreshRequested', this.setText);
    });
  }
});
{% endcodeblock %}

## Attach the components to the page

We have created our components but right now they are just isolated modules, so
we need to create a page and attach them to it so that we can interact with
them.

* **menuItem** is attached to each menu item.
* **menu** is attached to the whole menu.
* **menuContent** is attached to the menu content.

{% codeblock js/page/default.js lang:js %}
define(function (require) {
  'use strict';

  var menuItem = require('component/menu_item');
  var menuContent = require('component/menu_content');
  var menu = require('component/menu');

  return initialize;

  function initialize() {
    menu.attachTo('#menu');
    menuItem.attachTo('.menu-item');
    menuContent.attachTo('.menu-content');
  }
});
{% endcodeblock %}

## Wrap up

By now you should be able to open the `index.html` file on your browser and interact with your menu!

The way that our menu was created makes it easy to customize and update from
different places. Because of the "decoupleness" of our components we have a pretty
flexible and reusable menu.
