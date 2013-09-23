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

# Our components

# Getting Started

## Frameworks

* jasmine-flight
* mocha-flight

## Dos:

* Test event listeners and data
* Test event triggers and data
* Test initialization

## Don't:

* Test internal state of the component
* Test methods directly
