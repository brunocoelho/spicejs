[![Build Status](https://travis-ci.org/3den/spicejs.svg?branch=master)](https://travis-ci.org/3den/spicejs)

# What is Spice.js?

Spice is a super minimal (< 3k) and flexible MVC framework for javascript. Spice was built to be easily added to any existent application and play well with other technologies such as jQuery, pjax, turbolinks, node or whatever else you are using.

## Why yet an other MVC framework?

Currently all major MVC frameworks for javascript have a huge API and you end up writing more code to satisfy the the framework requirements than your own business logic, that leads to code that is tightly coupled to the framework and hard to reuse, sometimes even hard to update for new versions of the framework.

On Spice most of the code you write is pure javascript the framework API is minimal (just 5 methods) that helps you to make your code easier to reuse and test. The core of Spice is pure javascript (without any external dependency) that is fully unit tested and can be easily extended.

Spice.js was inspired by [the SOLID Principles](http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)), [jQuery](http://jquery.com/) and [Riot.js](https://github.com/muut/riotjs).

### The Good

* **Easy to learn**: Spice will few very straight forward for developers are familiar with jQuery and JavaScript.
* **Unobtrusive JavaScript**: Spice makes [progressive enhancement](http://en.wikipedia.org/wiki/Progressive_enhancement) simpler, you can load your views on the server and use Spice's controllers to add frontend features.
* **Powerful template engine**: Spice comes with `S.template` which creates a precompiled template that is supper fast, you can also write any javascript code in your template without having to learn different syntaxes.
* **Routes**: You can use routes to bind controllers and plugins, on pages where they are needed and you can have more than one route callback matching the same path.
* **Observables**: You can turn any javascript object, or function, into an observable that can listen and trigger events.
* **Supper Flexible**: You can use any javascript lib or jQuery plugin with spice, there is extensions for jQuery controllers, turbolinks and it is easy to write other extensions for the features you need.
* **Write once use everywhere**: Spice promotes reuse, it is easy to create generic controlers and models that can be reused in diferent parts of the same app or on diferent apps.
* **Testable**: Is very simple to test spice code, controllers are just a function, templates are just a string and observables are just objects. Spice also comes out of the box with a simple BDD framework.

### The Bad

* **To Flexible**: There is no restriction on your code structure so you can make a very good architecture with Spice or a very bad one.
* **To Young**: It is a new project so there is not much documentation yet.

### The Ugly

* **Using with other MVC Frameworks**: It is possible to use Spice with other javascript frameworks but that is not recommended since it could lead to a confusing code.


# Install

The best way to install `Spice.js`, and most js packages, is using bower:

```
$ bower install spicejs
```

By default bower will install packages in `./bower_components` so you can add spice and the extensions you need using:

```html
<script src="bower_components/spicejs/min.js"></script>
<!-- or "bower_components/spicejs/index.js" for the uncompressed version -->

<!-- Polyfill extension for compatibility with older browsers (IE 7) -->
<script src="bower_components/spicejs/ext/polyfill.js"></script>

<!-- Route extension for browser navigation and push state -->
<script src="bower_components/spicejs/ext/route_browser.js"></script>

<!-- Add other extensions as needed... -->
```

If you don't want to use bower you can just copy the files you need from https://github.com/3den/spicejs.

## Spice on Node.js

You can use Spice.js on node, to include spice do:

```js
var S = require("./bower_components/spicejs");

// now you can user spice on the backend
```

# S.observable(object)

This method turns any object or function into an `observable` by adding some methods for dealing with events properties and inheritance. The `observable` can be considered the M (Model) of MVC, if is deals with data and business logic.



```js
// Example of a a Search model
var Search = S.observable({
  query: undefined,
  page: 1,

  search: function(path, query) {
    var self = this;

    $.get(path, {
      format: "json",
      search: query
    }).done(function(data) {
      self.query = query;
      self.page = 1;

      // Triggers an event on the Search object.
      self.trigger("search", data, query);
    });
  }
});
```
Check the [unit tests](https://github.com/3den/spicejs/blob/master/test/lib/observable_test.js) for `S.observable`.

## observable.on(event, callback)

Observables can listen to events using the the `on` callback.

```js
Search.on("search", function(data, query) {
  console.log(data); // prints the data returned by the search
  console.log(data); // prints the query used to search
});
```

## observable.one(event, callback)

Does the same as `on` but the callback is altomatically removed after it is called the first time.

## observable.off(event, callback)

Allows you to remove event listeners.

```js
// Removes all callbacks
Search.off();

// Removes the 'search' callbacks
Search.off("search");

// Removes just the `someCallback` function 'search' callbacks
Search.off("search", someCallback);
```

## observable.trigger(event, arg1, arg2...)

Triggers an event, the arguments are passed to the callback lisetener.

```js
var data = [{id: 1, name: "Borderlands"}, {id: 2, name: "Doom 3"}]
  , query = "games";

// This will call all callbacks attached to the "search" event
// passing `data` and `query` as the arguments.
Search.trigger("search", data, query);
```

## observable.set(key, value)

Sets the value of a property and triggers the events "set" and `key`, passing

```js
var Order = S.observable({
  subtotal: 0,
  total: 0,
  shipping: 0
}).on("subtotal shipping", function(value, oldValue) {
  console.log("changed from " + oldValue + " to " + value);
  this.set("total", this.subtotal + this.shipping);
}).on("set", function(attr, value, oldValue) {
  console.log(attr + " changed from " + oldValue + " to " + value);
});

// Updates the subtotal to 10,
// the total will be altomatically set to 10
Order.set("subtotal", 10);

// Updates the shipping to 2.50,
// the total will be altomatically set to 12.50
Order.set("shipping", 10);
```

## observable.get(key)

Returns the value of a property.

```js
Order.total === Order.get('total');
```

## observable.create(properties)

Creates a new object based on the observable, Spice uses prototypal inheritance do changes made on the parrent object are inherited on the new one but changes on the child don't mess up with the parrent.

```js
var TaxableOrder = Order.create({
  tax: 0

}).off(
  // Removes the subtotal and shipping callbacks
  "subtotal shipping"

).on("subtotal shipping tax", function() {
  // Adds the new callback
  this.set("total", this.subtotal + this.shipping + this.tax);

});

// Setting the tax updates the total
TaxableOrder.set("tax", 1)
TaxableOrder.total === Order.total + 1

// Overriding a property dont touch the parent
TaxableOrder.set("subtotal", 5)
TaxableOrder.subtotal !== Order.subtotal
```


# S.template(text [, object])


# S.controller(name, callback)


## S.control(name, element [, options])


# S.route

The `S.route` method can do diverent things depending on what arguments are passed (following the jQuery sytle). The route matcher can have params `/search?q={query}` and wildcards `/users*`, and it is possible to have many one callbacks trigered for a given path.

Routes should be used to bind controllers, plugins or to trigger actions on a model. This will help to keep you code well organized and respond correctly to changes on the page.

Check the [unit tests](https://github.com/3den/spicejs/blob/master/test/lib/route_test.js) for `S.route`.

## S.route(callback)

This defines a generic route that will be called for all paths. This is usefull for binding plugins that will be used on all pages.

```js
S.route(function(params) {
  // binds select2 plugin to all `select` tags
  // http://ivaynberg.github.io/select2/
  $("select").select();

  // prints the path of the current page.
  console.log(params.path);
});
```

## S.route(path, callback)

The given `callback` will be called for all routes that match the given `path` expression.

```js

```

## S.route(object)

This sintax alows to set many route callbacks at once.

```js
// This code will do exactly the same as the previous example.
S.route({
  "/search*": function(params) {
    S.controll('search', document.getElementById("search-field"), {
      target: document.getElementById("search-results"),
      model: Search
    });
  },

  "/search?q={query}": function(params) {
    Search.search(params.path, params.query);
  }
});
```

## S.route(path [, triggersVisit = true])

Triggers all callbacks that match the given path, if `triggersVisit` is set to `false` it will NOT send a push state to update the browser.

```js
// Triggers all callbacks that match "/somepath"
// and also calls `route.trigger("visit", "/somepath")`
// that will send a push state to the browser updating
// the url if `ext/route_browser.js` was included.
S.route("/somepath");

// Triggers all callbacks that match "/somepath"
// but do NOT call `route.trigger("visit", "/somepath")`
// so the browser URL wont change
S.route("/somepath", false);
```

## S.route.update(path [, callsVisit = true])

Updates a portion the current current url and calls all callbacks that match the new path, if callsVisit is set to `false` it will update the URL without triggering the callbacks.

```js
// visits the "/search" and triggers all callbacks that match that route.
S.route("/search");

// Updates the current path to "/search?q=spice"
// and triggers all callbacks for visiting that route.
S.route.update("?q=spice");

// Updates the current path to "/search?q=spice#doc"
// WITHOUT triggering the callbacks for visinting that route
S.route.update("#doc", false);
```

# Extensions
