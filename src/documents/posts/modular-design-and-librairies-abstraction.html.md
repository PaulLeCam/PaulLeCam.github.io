---
title: Modular design and librairies abstraction
series: Client-side architecture, part 1
date: '5/27/2013'
layout: post
tags: ['client','architecture','modules','librairies','require','backbone','jquery']
abstract: An architecture made for abstraction and customization of librairies, assembled in a modular way to allow fast and simple evolutions.
---

## Background

About a year ago, I started to use [Aura](https://github.com/aurajs/aura) (Backbone-Aura at the time), a project by [Addy Osmani](http://addyosmani.com/) to present a clean structure for client-side code, following [Nicholas Zakas](http://www.nczonline.net/)' principles introduced in [this presentation](http://www.slideshare.net/nzakas/scalable-javascript-application-architecture).
I discovered it at a very convenient time for me, as I was starting to use [Require](http://requirejs.org/) and [Backbone](http://backbonejs.org) for all my client-side code, and Aura presented an interesting architecture for it, as well as implementing concepts from Zakas.

At the time, the project was in version 0.7 and used Backbone as a framework. The core was very simple to understand and use, so even though it was not very modular, it was very easy to customize for my applications needs.
Recent evolutions of Aura, with the release of version 0.9, unfortunately did not match my needs any more. The code feels more complex, and the choice to be used with any framework instead of only Backbone, if propbably interesting in a general manner, is more a constraint for me than anything.
So instead of continuing to customize Aura according to my needs, I went back to the basics of the concepts, and I ended up using a dead-simple architecture that just does the job.

## Using modules
This one is easy: [Require](http://requirejs.org/) does it for you.

Define your modules with their dependencies, assemble them in one or different files in your build process using [r.js optimizer](http://requirejs.org/docs/optimization.html), and dynamically load them in you code if you need.

## The core - abstracting librairies
I use [Lo-Dash](http://lodash.com/) and [jQuery](http://jquery.com/) in all my applications, so it felt like I would just need them anyway, but still I wanted to abstract them somehow beacuse, the same way I replaced underscore by Lo-Dash, I could use Zepto instead of jQuery. I needed a way to make these changes in a single place of my application and be sure other modules depending on it would still work as expected.

I ended-up splitting the librairies into core functionnalities I was using:

- **util**: common utilities, it is just and alias for Lo-Dash
- **template**: alias for Handlebars
- **dom**: a subset of jQuery DOM functions, such as `find()`, `data()` and events (`on()`, `off()`, `ready()`...)
- **http**: a subset of jQuery AJAX functions, such as `$.ajax()`, `$.get()`, `$.post()`...
- **promise**: an alias to jQuery's `$.Deferred()` and `$.when()`
- **events**: alias for Backbone's Events, with an `emit()` function aliasing to `trigger()` in order to match Node's EventEmitter
- **mvc**: alias for Backbone's Model, View and Collection
- **routing**: alias for Backbone's Router and `history.start()`

This way I could easily load dependencies in my modules in terms of functionalities instead of librairies, and already provide some form of sandboxing, as a module loading the `http` module would not have access to the `dom` functions, as it would if it had simply loaded jQuery, which is very useful to ensure the separation of concerns.

I also added some of my own abstraction to these core modules:

- **dev**: an alias to the console having to be explicitely activated, with polyfills for `time()` and `timeEnd()`
- **store**: a simple key/value store class with convenient methods
- **command**: an implementation of the [command pattern](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#commandpatternjavascript) using promises

All these very simple modules then served as a basis for creating a modular framework, easily extensible, that is presented in [part 2](/posts/make-your-own-framework/).
