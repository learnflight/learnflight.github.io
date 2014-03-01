---
layout: post
title: "How to integrate Flight into an existing Javascript application"
date: 2014-03-01 12:56
comments: true
categories: 
---

How to integrate Flight into and exisiting Javascript application

One of the best things about Flight is that it plays well with exisiting apps. This means that you don't need to migrate your whole app at once.
Because flight components can't communicate directly with the outside wolrd, gradually add new components without affecting the rest of our application.

For this example we are going to use a Color Raffle app. basically, this app consists of three parts.

1) Raffle button: Picks a random color whenever it is clicked
2) Color Stats: Each of the color stats will update everytime that color is selected from the raffle.
3) Histoy: Every time a color is selected, it will be added to the history wall.


NOTE: We will not go thought the process (setting up Flight and it depencencies)[], it is a really easy process.

### DEMO

{% codeblock application.js lang:js %}
$(function () {

  updateColorStat = function (color) {
    var $node = $("[data-color='" + color + "']");
    var count = parseInt($node.text()) + 1;
    $node.text(count);
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


Intentionally we use a really badly structured Javascript app, to show how can to intergate it with spagetti apps

An easy way to start integrating flight is to select one of the three components mentioned above(Raffle button, Color Stats and Histoy) and convert it into a Flight component.

Let's choose the color stats as a starting point. This is just a function that will update the count of the element which its `data-color` attribute matches the given color.

{% codeblock updateColorStat lang:js %}

updateColorStat = function (color) {
  var $node = $("[data-color='" + color + "']");
  var count = parseInt($node.text()) + 1;
  $node.text(count);
};

{% endcodeblock %}

In a new file `component/color_stat.js` lets create an empty flight component.

`
A Component is just a constructor with properties mixed into its prototype. It comes with a set of basic functionality such as event handling and Component registration.
``

In Flight A Component instance cannot be referenced directly; it communicates with other components via events.

{% codeblock components/color_stat.js lang:js %}

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

{% codeblock components/color_stat.js lang:js %}

define(function (require) {

  'use strict';

  var defineComponent = require('flight/lib/component');

  return defineComponent(colorStat);

  function colorStat() {
    this.updateColorStat = function (color) {
      if (this.$node.data('color') == color) {
        var count = parseInt($node.text()) + 1;
        this.$node.text(count);
      }
    }

    this.after('initialize', function () {

    });
  }
});

{% endcodeblock %}


Great! But how are we going to wire them together? This is the easiest part, we want to trigger a event every time the raffle trigger is clicked. 
So, lets replace the call to updateColorStat and trigger an event (see Flight events naming conventions for more info.)[] `uiColorSelected` and remove updateColorStat.

{% codeblock application.js lang:js %}
$('.Raffle-trigger').on('click', function () {
  var color = colors[Math.floor(Math.random()*colors.length)];
  $(this).trigger('uiColorSelected');
  updateHistory(color);
});
{% endcodeblock %}

Now lets add a listener to our Flight component.

{% codeblock components/color_stat.js lang:js %}
this.after('initialize', function () {
  this.on('uiColorSelected', this.updateColorStat);
});
{% endcodeblock %}

After attaching our component we are done.

