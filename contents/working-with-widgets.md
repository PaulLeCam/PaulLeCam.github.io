---
title: Working with widgets
series: Client-side architecture, part 3
date: "5/31/2013"
type: post
component: page
tags: ["client", "architecture", "framework", "require", "backbone", "jquery", "widgets"]
abstract: Define specific, loosely coupled units of UI components to integrate in your pages.
---

## Background

Managing loosely coupled pieces of UI components was one of the main reasons I started to use Aura, as I wrote in [part 1](/modular-design-and-librairies-abstraction/). Nevertheless, I did not need something as advanced as provided by Aura, all I needed was enhanced Backbone views that would be used as widgets.

The idea of widgets is that there would be simple, independent modules bound to DOM elements, that could easily be dynamically added and removed from the page, as well as being started and stopped at will.

## Widget class

The Widget class simply extends the View one by adding two main methods: `start` and `stop`.
```coffeescript
# When starting a widget for the first time, we render it.
# Then, later calls to the function will ensure events are bound.
start: ->
  @delegateEvents()
  unless @rendered
    @render()
    @rendered = yes

# Alias to `undelegateEvents()`
stop: ->
  @undelegateEvents()
```
In their very core, these methods deal with (un)binding the DOM events, but they should also be implemented to manage the mediator events they have to deal with. The `start` method also ensures the widget is actually rendered.

The full code for this class can be found [here](https://github.com/PaulLeCam/slob-client/blob/master/src/ext/framework.coffee#L132).

## Widgets management

The application does not directly deal with widgets, one of the main reason is that the widgets code should only be loaded when they are needed, allowing to load less startup code to run the application.

To manage the widgets lifecycle, a specific modules is loaded and used as a factory by the modules needing to interact with widgets, ensuring a loose coupling.

This module presents two main functions: `start(type, options)` and `stop(type, element)`, allowing the following behavior:

* When started for the first time, the Widget class is loaded (the `type` argument) and an instance is created and initialized with the `options` argument, that must at least contain an `el` property to be bound to a DOM element, and is then rendered.

* The widget can then be stopped by calling the `stop(type, element)` function, and restarted using the same arguments with the `start` function, that no longer needs the `options` argument but only the `element`.

* Because the widgets management module keeps track of all widgets, it is also possible to replace one of the arguments by `*` to start/stop at once all widgets of a certain type, or bound to a certain element.

The full code for this module can be found [here](https://github.com/PaulLeCam/slob-client/blob/master/src/ext/widgets.coffee), also containing functions to load and initialize widgets explicitly, in order to prepare them before being actually used, as well as functions to shut them down (emptying the element's content) or removing them.
