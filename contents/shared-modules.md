---
title: Shared modules
series: Optimizing client-server communication, part 1
date: "5/30/2013"
type: post
component: page
tags: ["client", "server", "architecture", "node", "backbone", "jquery"]
abstract: Share Backbone models, views and collections between the client and the server.
---

## Background

I can't remember when and where I read about the idea of Single-Page Applications for the first time, but I remember that I really enjoyed it.

After all, it was about transferring as much logic as possible from the server to the client, so thinking much more about the user interactions than the constraints of server-side developments.

I started to create prototypes that were mainly running client-side, mostly using the server as a REST API with a MongoDB or Redis database behind. As I wrote in [part 1 of my client-side architecture series](/modular-design-and-librairies-abstraction/), I used Aura at the time to create and manage widgets.

One thing that I really disliked using my applications, and I know it was a feeling I had as an *user* and not a developer, was that I always had to wait for the client code to load to actually being able to interact with the content. By that I mean that I had to wait until the client app loaded and sorted the items, before their corresponding views could be rendered and I could just *see* their content.

I felt very bad about it. Even though, as a developer, I could understand the whole process running in order to display the content I wanted, as an user I felt deeply frustrated seeing this loading animation, even for a few milliseconds.

As a developer, I could feel I was not doing something right. It took me some time until I realised that having created an "app", I had forgotten about the Web.

The Web is the platform, and it's great. One of the great things about it are URIs. It allows us to *directly* access a specific content.

On my smartphone, when I click on links to newspapers articles that direct me to the correspondig page, and this page asks me to download their application, they are just doing it wrong. This is a terrible user experience, even more because I know that *I already should be* reading the content I wanted to read. The application would just get in the way, because I don't need it. The Web got it right the first time with the URI, I am on the page containing the content I wanted to read.

Unfortunately, without realizing it, it is somehow what I had created with my Single-Page Applications rendering content on the client. I had broken the promise that the URI would display the requested content. Instead, it was loading a client app and asking it to do the work. It is what this very simple, short-living loader was reprensenting: a broken promise that the content was accessible there.

Nevertheless, I liked the ideas of Single-Page Applications, I liked to be thinking in terms of user interactions and events, but I did not want it to be more important than the content itself.

My idea was to be able to do both: being able to render content on the client or server, whatever is the most convenient or performant at the time. But because I was using Backbone for client-side developments, I also needed it on the server.

## Using Backbone on the server

Backbone has no problem running "as is" with Node.

Problems come when you need to render views, because jQuery isn't there. Actually, the DOM isn't there, there's no DOM in Node.

Good thing is, there's a Node module for that! It's called [jsdom](https://github.com/tmpvar/jsdom), and it makes it [very easy](https://github.com/PaulLeCam/node-slob/blob/master/src/jquery.coffee) to use jQuery thanks to it.

So it is pretty much what you need to use Backbone with Node, code made for the client just works with it.
I extended Backbone's classes according to my needs, most notably to be in sync with [what I did client-side](/posts/make-your-own-framework/).

The biggest difference is probably in the way of rendering views, as I implemented in the `renderer()` method (that has [its equivalent client-side](https://github.com/PaulLeCam/slob-client/blob/master/src/ext/framework.coffee#L125)), because instead of returning the view instance, it returns the full HTML element to be rendred:
```coffeescript
renderer: (html) ->
  @$el
    .attr("data-view", @cid)
    .html html
  template.renderSubViews @$el
  @el.outerHTML
```
Note that my customization of Backbone server-side is really made to match what I did client-side, as presented in [this article](/posts/make-your-own-framework/).

## Loading modules with Node and Require

The more I read about it, the more I discover ways of loading and packaging modules, both client-side and server-side.

npm is such a great tool I could not imagine not having it when I deal with Node. There is [browserify](https://github.com/substack/node-browserify) that seems to offer the same with client-side code, but I have never tried it.
With this project, I did not want to experiment new things, I only wanted to assemble things that I knew in a way I could easily use them, and my client-side code was using [RequireJS](http://requirejs.org/) to define and load modules.

I ended-up having a simple boilerplate for shared code, that would work both with Node and RequireJS:
```coffeescript
run = (mvc, exp) ->

  exp class User extends mvc.Model

    store: new mvc.Model.Store
    urlRoot: "/users"
    idAttribute: "_id"


if typeof exports is "undefined" # Browser
  define ["ext/framework"], (framework) ->
    run framework.mvc, (c) -> c

else # Node
  {framework} = require "slob"
  run framework.mvc, (c) -> module.exports = c
```
I hope the code is pretty self-explanatory: the `run` function is what should work on both sides, and the last `exp` argument is where it should return the code to be exposed. The following code then loads and exposes the dependencies depending on the context, client or server.

## Connect middleware

The purpose of this Node module was to offer a way to simply render on the server views that were originally meant to be created for the client.

In order to achieve this, we need to have other Backbone components (models and collections) working as well, because views generally use them.

At this point, I figured that I only wanted to interact with these components in the server views, so all I needed was to load a Connect middleware exposing these functions, as presented [here](https://github.com/PaulLeCam/node-slob/blob/master/src/middleware.coffee).

Then from the views, it is very easy to load stuff by simply calling the helper: `view("userInfos", {model: model("user", user_data)})`, assuming `user_data` is the data sent to the view. The middleware would load, instanciate and render the corresponding content.
