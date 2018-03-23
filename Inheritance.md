# Class Inheritance in Backbone

In nowadays, you'll modern JavaScript instead of setting up prototype chain manually. Nonetheless, it is helpful to understand Backbone's implementation & use ES6 with Backbone.

## `.extend` Method

When ES6 wasn't born, Backbone provides an `.extend` method on all of its classes, such that it is able to mimic traditional OO inheritance. 

Here's the `extend` method. It accepts 2 params:
- **protoProps** {Object}:
    an object containing instance methods
- **staticProps** {Object}:
    an object containing class methods.
    e.g. Child = Parent.extend({}, { foo: () => {} }); Child.foo();
```js
var extend = function(protoProps, staticProps) {
    var parent = this;  // Whoever calls this function.
    var child;          // the child constructor.

    // `child` must be a function. If `protoProps` provides no
    // `constructor` function, wrap it in a function
    if (protoProps && _.has(protoProps, 'constructor')) {
        child = protoProps.constructor;
    } else {
        child = function() { return parent.apply(this, arguments); }
    }

    _.extend(child, parent, staticProps);

    // Makes child.prototype.__proto__ = parent.prototype
    child.prototype = _.create(parent.prototype, protoProps);
    child.prototype.constructor = child;

    // Just another shortcut.
    child.__super__ = parent.prototype;
    return child;
};

// Assign to them all.
Model.extend = Collection.extend = Router.extend = View.extend = History.extend = extend;
```
This method works pretty straight forward:

1. a parent class(e.g. `Backbone.Model`, etc.) calls `.extend` with a `protoProps` object, and a `staticProps` object:

```js
    parent.extend(protoProps, staticProps);
```

2. `.extend` method extracts `constructor` property from `protoProps` to be the child's constructor. If there isn't a `constructor` property, create one.

3. Set up child's static methods.

4. Set up prototype chain, that is, 
```js
    child.prototype = _.create(parent.prototype, protoProps);
```    

`_.create` will clone `protoProps` and set  `parent.prototype` to the clone object's proto chain. This is equivalent to:

```js
    child.prototype = {};
    child.prototype.__proto__ = parent.prototype;
    for (var key in protoProps) {
        child.prototype[key] = protoProps;
    }
```
Then all child's instances will be able to access parent's instance methods.

## Constructor
A thing worth noting is, **if you provide a constructor, make sure your constructor calls parent's constructor function, with the context of a child instance.** Here's an example(this example is not a good design):

```js
// Not good.
var MusicModel = Backbone.Model.extend({
    constructor: function(attrs) {
        attrs = attrs || {};
        this.title = attrs.title || null;
        this.artist = attrs.artist || null;
        // Make sure you call it.
        // `this` references to an instance of MusicModel.
        Backbone.Model.apply(this, arguments);
    }
});
```
Or if you're using ES6 style, you just make a call to `super`:

```js
// Backbone doesen't support ES6 module. Take CommonJS instead.
const { Model } = require('Backbone');

class MusicModel extends Model {
    constructor(attrs, options) {
        super(attrs, options);
        attrs = attrs || {};

        this.title = attrs.title || null;
        this.artist = attrs.artist || null;
    }
};
```

In rare cases you want to provide a constructor, unless you want to manage some instance properties on your own(sometimes you may want to). That means, if you try to use the following methods, these self-managed properties wouldn't be taken into account:

- set - It might also cause problem in "change" event's handler
- get
- toJSON
- has
- Any other method operate on `.attributes` property

Instead, you should set instance properties when you "`new`" an instance:

```js
var MusicModel = Backbone.Model.extend({
    // If you want every instance to have a shape,
    // set up default attribute values.
    defaults: {
        title: null,
        artist: null
    }    
});

// When you "new" an instance
var song = new MusicModel({
    title: "Single",
    artist: "Ne-Yo"
});

// When you want to get or set one of them
var oldTitle = song.get("title");
song.set({title: "Mad"});
```
As an alternative, write ES6 in this way:

```js
const { Model } = require('Backbone');

class MusicModel extends Model {
    constructor(attrs, options) {
        // Make a call to Model to obtain `this`.
        super(attrs, options)
    }

    initialize() {
        /* ... */
    }

    defaults() {
        title: null,
        artist: null
    }
};
```