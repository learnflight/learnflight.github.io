---
layout: post
title: "Testing a Flight component"
date: 2013-09-20 09:53
comments: true
published: false
categories:
---

We are going to go through an example of testing a small set of components. In this case we are going to test `flight-dropdown`  a minimal component that implements  dropdown.

Testing components in Flight is damn fun. This is because by nature they are fully decoupled from the rest of your application.

As of today the best to way of testing Flight is with `jasmine-flight` or `mocha-flight`, which are extensions to Jasmine and Mocha respectively.

For this post we are going to use the simplest to-do app. With this app the user is only able to add new items to the list, but it is awesome to ilustrate the testing needed for a Flight app.

[View source](https://github.com/rogeliog/learn-flight-navigation-menu-demo/tree/master/app) or [Live demo](http://rogeliog.github.io/learn-flight-navigation-menu-demo).

# Testing principles

Flight components are, by nature, completely decoupled from each other.
Because Flight components can only comunicate via events we should focus on testing the events and not the methods indise the components.

For the general case:

#### Do test

* Event listeners
* Event triggers
* defaultAttrs

#### Don't test

* Methods
* Internal state

## Our components

#### UI components:
* `ui/items.js`: It displays the to-do items list.
* `ui/compose_box.js`: Where the user types the new to-do items

#### Data components:
* `data/items.js`: Simulates a backend. It saves and serves to-do items.

## Testing data components

Some imporants things to remember about data components are:

* Don't test anything related with the DOM.
* They shoudn't know about the UI structure.

{% codeblock data/items.js lang:js %}
define(function (require) {

  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(items);

  function items() {
    this.defaultAttrs({
      todoItems: []
    });

    this.createItem = function (e, data) {
      this.attr.todoItems.push(data.item);
      this.triggerDataItems();
    }

    this.triggerDataItems = function () {
      this.trigger('dataItems', { items: this.attr.todoItems });
    }

    this.after('initialize', function () {
      this.on('uiNewItemAction', this.createItem);
      this.triggerDataItems();
    });
  }

});
{% endcodeblock %}

This components is doing basically two things.

* Listens to the `uiNewItemAction` event, adds the new item to the list of todo items and triggers `dataItems` to notify thatthe todo items were updated.
* Triggers the `dataItems` after initialization to notify components about the initial todo items.

These are the only two things that we are going to test.

{% codeblock spec/ui/items.spec.js lang:js %}
'use strict';

describeComponent('component/data/items', function () {

  describe('Initialization', function () {
    it('triggers dataItems with the initial todo items', function () {
      var eventSpy = spyOnEvent(document, 'dataItems');
      setupComponent({ todoItems: ["old item"] });

      expect(eventSpy).toHaveBeenTriggeredOnAndWith(document, {
        items: ["old item"]
      });
    });
  });

  describe('Listens to uiNewItemAction', function () {
    it('and triggers dataItems with current item and the new one', function () {
      setupComponent({ todoItems: ["old item"] });

      var eventSpy = spyOnEvent(document, 'dataItems');
      this.$node.trigger('uiNewItemAction', {item: "new item"});

      expect(eventSpy).toHaveBeenTriggeredOnAndWith(document, {
        items: ["old item", "new item"]
      });
    });
  });
});
{% endcodeblock %}
