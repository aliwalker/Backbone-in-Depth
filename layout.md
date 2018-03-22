## Layout
Backbone uses a module pattern to wrap everything up in an IIFE, in order to protect globall polution:

```js
(function(factory) {

    // detect environment. self(window) for browser, global for Node.js
    var root = (typeof self == 'object' && self.self === self && self) ||
               (typeof global == 'object' && global.global === global && global);

    // detect which module scheme is used. First AMD.
    if (typeof define === 'function' && define.amd) {
        // Dependency injection
        define(['underscore', 'jquery', 'exports'], function(_, $, exports) {
            root.Backbone = factory(root, exports, _, $);
        });
      // CommonJS, might be using browserify.
    } else if (typeof exports !== 'undefined') {
        var _ = require('underscore'), $;
        try { $ = require('jquery'); } catch (e) {}
        factory(root, exports, _, $);
      // script tag.
    } else {
        root.Backbone = factory(root, {}, root._, (root.jQuery || root.Zepto || root.ender || root.$));
    }

})(function(root, Backbone, _, $){
    var previousBackbone = root.Backbone;
    ...
    Backbone.noConflict = function() {
        root.Backbone = previousBackbone;
        return this;
    };
});
```

If you've read the source code of jQuery, it is familiar with you.
The initial setup is wrapped in an IIFE, which accepts a factory function for constructing Backbone. Before making a call to this factory function, the IIFE first detech environment to set up the right root object, then check to see which module scheme is used. There are 3 available schemes, AMD, CommonJS or script-tag.

`noConflict` function is pretty straight forward. Before overwritting `root.Backbone`, store it in a local variable, such that when there's a conflict, you can:

```js
// Bb is the Backbone object we're talking about.
var Bb = Backbone.noConflict();
```

## addUnderscoreMethods function
Used for adding underscore methods to `Backbone.Model` & `Backbone.Collection`.

```js
var addUnderscoreMethods = function(Class, methods, attribute) {
    _.each(methods, function(length, method) {
        if (_[method]) Class.prototype[method] = addMethod(length, method, attribute);
    })
}
```

The author believes that using `apply` on a function can be slow, thus if the number of params is known in advance, we can avoid `apply`. The `methods` param to `addUnderscoreMethods` is an object in the shape of:

```js
methods = {
    "each": 3,
    ...
}
```