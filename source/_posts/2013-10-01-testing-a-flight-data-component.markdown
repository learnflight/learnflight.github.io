---
layout: post
title: "Testing a Flight data component"
date: 2013-10-01 09:53
comments: true
published: true
categories:
---

Testing components in Flight is damn fun. This is because, by nature, they are fully encapsulated and decoupled from the rest of your application.

As of today the best way of testing Flight is with [jasmine-flight](https://github.com/flightjs/jasmine-flight) or [mocha-flight](https://github.com/flightjs/mocha-flight), which are extensions to Jasmine and Mocha respectively.

For this post we are going to use a really simple to-do app. A to-do app will generally need a component that handles the data, which has the responsibility of adding/removing items, and notifying other components about that.

## Flight testing principles (optional)

Flight components communicate via events, so we are going focus on testing only events and not the methods inside the component.

Here is a general rule of thumb for testing a Flight component.

<!-- more -->

#### Do test

* **Event listeners**: Any event that the component is listening to.
* **Event triggers**: Any event that the component can trigger.
* **defaultAttrs**: Customization via defaultAttrs.
* **Initialization**: Special events that are triggered after initialization.

#### Don't test

* **Methods**: The internal methods that are called when listening or triggering events.
* **Internal state**: Any component state that is not part of the `this.attr` property.

### Setup

In this case we are going to be using jasmine-flight, you might want to go and [set it up](https://github.com/flightjs/jasmine-flight#getting-started).

## Testing data components

Some important things to remember about data components are:

* Don't test anything related with the DOM.
* They shouldn't know about the UI structure.

We have a `data/items.js` which simulates a backend, it adds/removes to-do items and serves them.

This components is basically doing three things.

* Listens to the `uiNewItemAction` event, adds the new item to the list and triggers `dataItems` to notify that the items were updated.
* Listens to the `uiRemoveItemAction` event, removes the selected item from the list and triggers `dataItems` to notify that the items were updated.
* Triggers the `dataItems` after initialization to notify components about the initial to-do items.

For the sake of simplicity we are going to save the to-do items in an array, but our tests will never know about that, so we can modify it to save on local-storage or in the backend without altering the tests too much.

*NOTE: For a more robust storage component see [flight-storage](https://github.com/cameronhunter/flight-storage).*

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

    this.removeItem = function (e, data) {
      this.attr.todoItems = this.attr.todoItems.filter(function (item) {
        return item != data.item;
      });
      this.triggerDataItems();
    }

    this.triggerDataItems = function () {
      this.trigger('dataItems', { items: this.attr.todoItems });
    }

    this.after('initialize', function () {
      this.on('uiNewItemAction', this.createItem);
      this.on('uiRemoveItemAction', this.removeItem);
      this.triggerDataItems();
    });
  }

});
{% endcodeblock %}

{% codeblock spec/data/items.spec.js --- TEST FILE lang:js %}
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

  describe('Listeners', function () {

    beforeEach(function () {
      setupComponent({ todoItems: ["item 1", "item 2"] });
    });

    describe('Listens to uiNewItemAction', function () {
      it('and triggers dataItems with current item and the new one', function () {
        var eventSpy = spyOnEvent(document, 'dataItems');
        this.component.trigger('uiNewItemAction', {item: "new item"});

        expect(eventSpy).toHaveBeenTriggeredOnAndWith(document, {
          items: ["item 1", "item 2", "new item"]
        });
      });
    });

    describe('Listens to uiRemoveItemAction', function () {
      it('and triggers dataItems without the selected item', function () {
        var eventSpy = spyOnEvent(document, 'dataItems');
        this.component.trigger('uiRemoveItemAction', {item: "item 2"});

        expect(eventSpy).toHaveBeenTriggeredOnAndWith(document, {
          items: ["item 1"]
        });
      });
    });
  });

});
{% endcodeblock %}

Lets break down this test file and walkthough it.

{% codeblock describeComponent lang:js %}
'use strict';

describeComponent('component/data/items', function () {
  // more code
});
{% endcodeblock %}

[describeComponent](https://github.com/flightjs/jasmine-flight#components) receives the path of the component that is going to be tested.


{% codeblock Initialization lang:js %}
describe('Initialization', function () {
  it('triggers dataItems with the initial todo items', function () {
    var eventSpy = spyOnEvent(document, 'dataItems');
    setupComponent({ todoItems: ["old item"] });

    expect(eventSpy).toHaveBeenTriggeredOnAndWith(document, {
      items: ["old item"]
    });
  });
});
{% endcodeblock %}

[spyOnEvent](https://github.com/flightjs/jasmine-flight#event-spy) works for events in the same way that Jasmine's [spyOn](http://pivotal.github.io/jasmine/#section-Spies) works for functions.

[setupComponent](https://github.com/flightjs/jasmine-flight#setupcomponent) creates an instance of the component, it also sets the instance to `this.component`. You can pass an object to `setupComponent` and it will be set as the `defaultAttrs`.

For this test we are spying on the event that we expect to be triggered, instantiating the components and making sure that the event was triggered with the appropriate data (the items in the to-do list).


{% codeblock Adding new to-do items lang:js %}
describe('Listeners', function () {

  beforeEach(function () {
    setupComponent({ todoItems: ["item 1", "item 2"] });
  });

  describe('Listens to uiNewItemAction', function () {
    it('and triggers dataItems with current item and the new one', function () {
      var eventSpy = spyOnEvent(document, 'dataItems');
      this.component.trigger('uiNewItemAction', {item: "new item"});

      expect(eventSpy).toHaveBeenTriggeredOnAndWith(document, {
        items: ["item 1", "item 2", "new item"]
      });
    });
  });

  // more code
});
{% endcodeblock %}

This test makes sure that the user is able to successfully add new items to the list. We are going to be sharing the call to `setupComponent` for both tests.

For this test we are spying on the `dataItems` event and expecting it to be triggered with the two original items (item 1, item 2) and the "new item" after the `uiNewItemAction` event is triggered.

{% codeblock Removing to-do items lang:js %}
describe('Listeners', function () {

  beforeEach(function () {
    setupComponent({ todoItems: ["item 1", "item 2"] });
  });

  // more code

  describe('Listens to uiRemoveItemAction', function () {
    it('and triggers dataItems without the selected item', function () {
      var eventSpy = spyOnEvent(document, 'dataItems');
      this.component.trigger('uiRemoveItemAction', {item: "item 2"});

      expect(eventSpy).toHaveBeenTriggeredOnAndWith(document, {
        items: ["item 1"]
      });
    });
  });
});
{% endcodeblock %}

This test is opposite of the previous one, it test the removal of items from the list.


# Conclusion

Note how we never tested any method inside the component; we don't know anything about its internal structure. This makes refactoring the internals really simple, because no test is bounded to implementation. Actually we don't even know or care where the items are being saved, they might be saved on localStorage, cookies, in-memory, etc.
