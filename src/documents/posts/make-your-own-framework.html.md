---
title: Make your own framework
series: Client-side architecture, part 2
date: '5/28/2013'
layout: post
tags: ['client','architecture','framework','require','backbone','jquery']
abstract: Making your own framework by assembling and customizing components is easy, just create what you need.
---

## Background

In [part 1](/posts/modular-design-and-librairies-abstraction/), I presented a way to simply extract functions from different librairies, normalize and expose them in a modular way.
The goal was to abstract these underlying librairies, in order to customize their behavior for the needs of the applications.

## Extending Backbone and Handlebars
[Backbone](http://backbonejs.org) is a great framework presenting Model, View and Collection classes, and a router. I use it along with [Handlebars](http://handlebarsjs.com) for templating.

[This article](http://backstage.soundcloud.com/2012/06/building-the-next-soundcloud/) from the SoundCloud engineering team presents some great ideas on how to push them further, most notably on two aspects I wanted to implement: sharing models between views, and rendering sub-views.

### Sharing models
The idea of sharing models between views is that a specific model should not require to be instanciated multiple times.

If you have an "user" model, it may be used by different views on the page, that may require different attributes from this model. Instead of creating the user model multiple times, we can ensure that only one exists for a specific `id`, and contains all the attributes we may need.

Here is how I extend the Model constructor:
```coffeescript
# Each Model class must have its own store of instances
store: new Store

# When we instanciate a model, we check in the store if it is not already present.
# If this is the case, we silently update its data.
constructor: (params = {}) ->
  if (id = params.id or params.cid) and self = @store.get id
    self.set params, silent: yes
    return self

  super params
  key = @id ? @cid
  @store.set key, @
```
This is very basic: every Model contains an internal registry, `@store`, that contains its instances identified by `id` or `cid`. If the instance is present in the registry, it gets updated with possible new data and directly returned. Otherwise, it is normally created and added to the registry.

As stated in the SoundCloud engineering blog post, it is an application of the [factory pattern](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#factorypatternjavascript) directly in the constructor, allowing to simply use `user = new User(id: 123)` and actually retrieve the user with all the attributes previously set.

Note that, again as stated in the article, this may not be very memory-efficient and a more advanced caching system would be interesting to auto-delete unused instances after some time.

### Rendering sub-views
A limit often presented with Backbone is the ability to simply render sub-views, meaning for example the ability to render each item in a list when rendering the list, and ensuring the events are correctly bound.

This is why I augment Handlebars (the `template` object in the following code) with custom functions to ensure this goal:
```coffeescript
# Local store for subviews
subviews = new Store

# Add a view to the local store and return a DOM element that we can identify
template.addSubView = (view) ->
  subviews.set view.cid, view
  new template.SafeString "<view data-cid=\"#{ view.cid }\"></view>"

# Return the DOM element of a stored view and delete it from the store
template.renderSubView = (cid) ->
  if view = subviews.get cid
    subviews.delete cid
    view.render().el
  else ""

# Render all subviews present in a DOM element identified by a jQuery object
template.renderSubViews = ($el) ->
  $el.find("view").each (i, view) ->
    $view = $el.find view
    $view.replaceWith template.renderSubView $view.data "cid"
```
First, we need to store the sub-view of the item and return a placeholder `view` element for the list to render.

Then, once the list is rendered, we call `renderSubViews()` on its element to iterate through the placeholders and replace them with the actual item content.

This last function is then used in a method added to the View class:
```coffeescript
# The `renderer()` set the HTML content for the element and render eventual associated subviews
renderer: (html) ->
  @$el
    .attr("data-view", @cid)
    .html html
  template.renderSubViews @$el
  @
```

Here is a full (simple but not tested) example:
```coffeescript
class ItemView extends View
  
  tmpl: template.compile """
    <a href="{{uri}}">{{label}}</a>
    """

  events:
    "click": "onClick"

  onClick: ->
    console.log "clicked item", @options

  render: ->
    @renderer @tmpl @options

template.registerHelper "renderItem", (data = {}) ->
  template.addSubView new ItemView data

class ListView extends View
  
  tmpl: template.compile """
    <ul>
      {{#each items}}
        <li>{{renderItem this}}</li>
      {{/each}}
    </ul>
    """

  el: "#myList"

  render: ->
    @renderer @tmpl @options

items = [
  {uri: "uri1", label: "label1"}
  {uri: "uri2", label: "label2"}
  {uri: "uri3", label: "label3"}
]
new ListView({items}).render()
```
Note that there are tons of ways to render sub-views and I have no idea which is the best.

I choose this one because I did not want to have a strong coupling between `ListView` and `ItemView`, this way it is the `renderItem` helper that defines the view to bind. It is then possible either to change the view in the helper, or the helper called in the template, instead of having to deal with the View classes.

In a future part 3 of this series, I will focus on creating and managing widgets as simple UI components in a page.
