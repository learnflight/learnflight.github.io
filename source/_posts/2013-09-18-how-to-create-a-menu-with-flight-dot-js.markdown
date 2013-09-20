---
layout: post
title: "How to create a menu with Flight.js"
date: 2013-09-18 15:56
comments: true
categories:
---

We are going to create a simple navigation menu with [Flight.js](http://flightjs.github.io/). The purpose of this post is to illustrate a practical Flight.js example. Here are the [docs](https://github.com/flightjs/flight/tree/master/doc) and the [github page](https://github.com/flightjs/flight) if you want to check them out.

{% img center /images/navigation-menu.png %}

[View source](https://github.com/rogeliog/learn-flight-navigation-menu-demo/tree/master/app) or [Live demo](http://rogeliog.github.io/learn-flight-navigation-menu-demo).

<!-- more -->

# Initial Setup

We need to setup some boring stuff to be sure that we all have the same beginning state. The fun stuff starts in the next section.

Before we start be sure to have installed [npm](https://npmjs.org/) and [bower](http://bower.io/). Lets create a new folder for our project. Inside our folder lets setup our project. We are going to use the [generator-flight](https://github.com/flightjs/generator-flight) yeoman generator to create the basic scaffold for our application.

This will install the appropiate yeoman generator and now we can create our app.

{% codeblock lang:bash %}
# might need sudo
npm install -g generator-flight
{% endcodeblock %}

{% codeblock lang:bash %}
# NOTE: we don't need Normalize.css or Bootstrap.css
yo flight
{% endcodeblock %}

Once we have the basic scaffold we can add some markup and styles

### Add the markup

{% codeblock app/index.html lang:html %}
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
{% endcodeblock %}


### Paste some CSS

We also need to paste some CSS to make it a look a bit sexier.

{% codeblock app/css/main.css lang:css %}
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
}

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

### Run the app

Lets run the app to be sure that everything is wired up properly.

{% codeblock lang:bash %}
# might need sudo
npm install -g http-server
http-server app
{% endcodeblock %}

Visit [localhost:8080](http://localhost:8080)

# Create the components

Now we can start creating our components. In this case we are going to create
three of them.

* One for the *menu title*, another for the *menu content* and third one for *generating the data* that will be presented in the menu

The first two components will be **ui components**, this means that we are going
to attach them to a DOM element and they will manipulate it.

The third component is a **data component**. This component doesn't know about the DOM state or the menu.
It will only be generating the "backend" response with the appropiate content for
a given section.

**ui/menu_item.js** This is the most complex component. It only knows about the menu title

  * Listens to clicks on the `.menu-items` and triggers a `uiNeedsMenuSection`
  event to notify that we need to change the menu section.
  * Listens to the `dataMenuSection` event sent from a data component, adds the appropiate class to highlight the `.menu-item`

{% codeblock app/js/component/ui/menu_title.js lang:js %}
define(function (require) {
  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(menuItems);

  function menuItems() {
    this.defaultAttrs({
      itemSelector: '.menu-item'
    });

    this.highlightSection = function (e, data) {
      this.$node.removeClass().addClass(data.section);
    }

    this.requestSectionChange = function (e) {
      this.trigger('uiNeedsMenuSection', {
        section: $(e.target).text().toLowerCase()
      });
    }

    this.after('initialize', function () {
      this.on('click', {
        itemSelector: this.requestSectionChange
      });

      this.on(document, 'dataMenuSection', this.highlightSection);
    });
  }
});
{% endcodeblock %}

**ui/menu_content.js** It only knows how to replace the content given a markup

  * Listens to the `dataMenuSection` event, updates the content with the appropriate markup notify that content changed.

Note how it doesn't know anything about the `ui/menu_title.js` component or
that the content needs to be changed because a menu section was clicked.

{% codeblock app/js/component/ui/menu_content.js lang:js %}
define(function (require) {
  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(menuContent);

  function menuContent() {
    this.setText = function (e, data) {
      this.$node.html(data.markup);
    }

    this.after('initialize', function () {
      this.on(document, 'dataMenuSection', this.setText);
    });
  }
});
{% endcodeblock %}

**data/menu_section.js** This component knows how to get the appropiate data for any section.

  * Listens to the `uiNeedsMenuSection` event and triggers a `dataMenuSection` to
  notify components that there is new content available for the menu.

In this component you can create a `renderSection` function that will render a [mustache](http://mustache.github.io/) template
for that given section or even get some some data from the backend.

{% codeblock app/js/component/data/menu_section.js lang:js %}
define(function (require) {
  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(menuSection);

  function menuSection() {
    this.changeSection = function (e, data) {
      this.trigger('dataMenuSection', {
        section: data.section,
        markup: '<b>' + data.section + '</b>'
      });
    }

    this.after('initialize', function () {
      this.on('uiNeedsMenuSection', this.changeSection);
    });
  }
});
{% endcodeblock %}

## Attach the components to the page

We have created our components but right now they are just isolated modules, so
we need to create a page and attach them to it so that we can interact with
them.

* **menuTitleUI**: attached to each menu item.
* **menuContentUI**: attached to the menu content.
* **menuSectionData**: it is a data component so it will be attached to the document.

{% codeblock app/js/page/default.js lang:js %}
define(function (require) {
  'use strict';

  var menuTitleUI = require('component/ui/menu_title');
  var menuContentUI = require('component/ui/menu_content');
  var menuSectionData = require('component/data/menu_section');

  return initialize;

  function initialize() {
    menuTitleUI.attachTo('#menu');
    menuContentUI.attachTo('.menu-content');

    menuSectionData.attachTo(document)
  }
});
{% endcodeblock %}

## Wrap up

By now you should be able to run your server and interact with your menu!

It is important to notice how decoupled each component is from each other.
Because that "decoupleness" of our components we have a pretty flexible and reusable menu.


*NOTE: English is not my native language, I'll be more than happy to receive any grammar or spelling corrections :)*
