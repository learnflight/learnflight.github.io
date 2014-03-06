---
layout: post
title: "How to integrate Flight into an existing Javascript application"
date: 2014-03-01 12:56
comments: true
categories:
---

How to integrate Flight into an exisiting Javascript application.

One of the best things about Flight is that it plays well with exisiting apps. This means that you don't need to migrate your whole app at once.
Because flight components can't communicate directly with the outside wolrd, you can gradually migrate new components without affecting the rest of the application.

For this example we are going to use a Color Raffle application, this app consists of three basic parts.

1. **Raffle button:** Picks a random color whenever it is clicked.
2. **Color Stats:** Each of the color stats will update everytime that given color comes out of the raffle.
3. **Histoy:** Every time a color is selected, it will be added to the history wall.

NOTE: We will not go thought the process (setting up Flight and it depencencies)[], but it is a really easy process.

### DEMO PENDING ###

{% codeblock application.js lang:js %}
$(function () {

  updateColorStat = function (color) {
    $('.tweet-stat').each(function () {
      var $node = $(this);
      var count = parseInt($node.text());

      if (this.$node.data('color') == data.color) {
        this.$node.text(count + 1);
      } else {
        this.$node.text(count - 1);
      }
    });
  };

  updateHistory = function (color) {
    var $history = $('.history');
    $history.append('<li>' + color + '</li>');
  };

  var colors = ['red', 'green', 'blue'];
  $('.Raffle-trigger').on('click', function () {
    var color = colors[Math.floor(Math.random()*colors.length)];
    updateColorStat(color);
    updateHistory(color);
  });
});

{% endcodeblock %}


Intentionally we use a really badly structured Javascript app. An easy way to start integrating flight is to select one of the three components mentioned above(Raffle button, Color Stats and Histoy) and convert it into a Flight component.

Let's choose the color stats as a starting point. This is just a function that will update the count of the element which its `data-color` attribute matches the given color.

{% codeblock updateColorStat lang:js %}

updateColorStat = function (color) {
  $('.tweet-stat').each(function () {
    var $node = $(this);
    var count = parseInt($node.text());

    if (this.$node.data('color') == data.color) {
      this.$node.text(count + 1);
    } else {
      this.$node.text(count - 1);
    }
  });
};
{% endcodeblock %}

In a new file `component/color_stat.js` lets create an empty flight component.

{% blockquote %}
A Component is just a constructor with properties mixed into its prototype. It comes with a set of basic functionality such as event handling and Component registration.
{% endblockquote %}

In Flight a component instance cannot be referenced directly; it communicates with other components via events.

{% codeblock components/color_stats.js lang:js %}

define(function (require) {

  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(colorStat);

  function colorStat() {
    this.after('initialize', function () {

    });
  }
});

{% endcodeblock %}

Now that we have our empty component, lets copy the `updateColorStat` logic into the component, and update some references

## Color Stats with updateColorStat Flight Component ##

{% codeblock components/color_stats.js lang:js %}

define(function (require) {

  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(colorStat);

  function colorStat() {
    this.updateColorStat = function (event, data) {
      var count = parseInt($node.text());

      if (this.$node.data('color') == data.color) {
        this.$node.text(count + 1);
      } else {
        this.$node.text(count - 1);
      }
    }

    this.after('initialize', function () {

    });
  }
});

{% endcodeblock %}

Great! But how are we going to wire them together? This is the easiest part, we simply want to trigger an event every time the raffle button is clicked.
So, lets replace the call to `updateColorStat` and trigger an event (see Flight events naming conventions for more info.)[] `uiColorSelected` and remove updateColorStat.

{% codeblock application.js lang:js %}
$('.Raffle-trigger').on('click', function () {
  var color = colors[Math.floor(Math.random()*colors.length)];
  $(this).trigger('uiColorSelected', { color: color });
  updateHistory(color);
});
{% endcodeblock %}

Now lets add a listener to our Flight component.

{% codeblock components/color_stat.js lang:js %}
this.after('initialize', function () {
  this.on(document, 'uiColorSelected', this.updateColorStat);
});
{% endcodeblock %}

We are going to attach our component to each of the tweet stats element and done.

{% codeblock page/default.js lang:js %}

var TweetStat = require('components/color_stat');

TweetStat.attachTo('.tweet-stat');

{% endcodeblock %}

By now we are done migrating our first component, lets continue with the history component. We are going to follow the same steps

1) Create an empty component.

{% codeblock components/history.js lang:js %}

define(function (require) {

  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(history);

  function history() {
    this.after('initialize', function () {

    });
  }
});

{% endcodeblock %}

2) Move logic into the component.

{% codeblock components/history.js lang:js %}

define(function (require) {

  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(history);

  function history() {
    updateHistory = function (color) {
      this.$node.append('<li>' + color + '</li>');
    };

    this.after('initialize', function () {

    });
  }
});

{% endcodeblock %}

3) Listen to desired event.

{% codeblock components/history.js lang:js %}
  this.after('initialize', function () {
    this.on(document, 'uiColorSelected', this.updateHistory);
  });
{% endcodeblock %}

4) Attach component to element

{% codeblock page/default.js lang:js %}

var TweetStat = require('components/color_stat');
var Histoy = require('components/histoy');

TweetStat.attachTo('.tweet-stat');
Histoy.attachTo('.histoy');

{% endcodeblock %}

5) Remove logic from legacy code.

{% codeblock application.js lang:js %}
$(function () {
  var colors = ['red', 'green', 'blue'];

  $('.Raffle-trigger').on('click', function () {
    var color = colors[Math.floor(Math.random()*colors.length)];
    $(this).trigger('uiColorSelected', { color: color });
  });
});

{% endcodeblock %}

By now we have already moved two of the three components, I will let you do the last one on your own, but it basically the same steps.

So, what have we achieved? We have divided the responsabilities accross different components that are completely decoupled from each other. Scaling this app is now easier and more maintainable due to the modular nature of Flight.

The three components are ui components, but in the next post we are going to see how to extract a data component that handles all the data logic and provide it to the ui components.
