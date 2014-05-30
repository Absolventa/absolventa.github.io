---
layout: post
title: "JSDoc and the Revealing Module Pattern"
date: 2014-03-25 16:35
author: Carsten Zimmermann
googleplus: "https://plus.google.com/u/0/101728608052847168461"
comments: true
tags:
  - javascript
  - module
  - jsdoc
teaser: "Generating documentation for your JavaScript with JSDoc can be tricky for a combination of the Revealing Module Pattern and self-executing Javascript functions. This article shows how JSDoc can be convinced to parse the code anyway."
---

Before the grand [v1.0.0](http://en.wikipedia.org/wiki/Beta_version#Beta) of an internal asset pipeline gem  with lots of extracted javascript from our various Rails apps, I wanted to see what our API documentation looked like in colourful HTML.

Much to my surprise, [JSDoc](http://usejsdoc.org/) didn't show a thing when I
ran it over our .js files: We use the [Revealing Module
Pattern](http://www.klauskomenda.com/code/javascript-%20programming-patterns/#revealing)
with self-executing code to define our Javascript and JSDoc had some trouble
parsing it.

Documentation on how to fix this was scarce - everyone seemed to have a slightly different use case – so here's what we came up with after some trial and error runs with various JSDoc keywords.

Our Javascript looks like this:

{% highlight javascript %}
// In file: namespace.js
(function() {
    window.Absolventa = window.Absolventa || {};
}());

// In file: modules/urlify.js
(function() {
    "use strict";
    Absolventa.Urlify = (function() {
        var init;

        /**
         * @param {string} foo
         */
        init = function(foo) {
          // Magick!
        };

        return {
          init : init
        };
    }());
}());
{% endhighlight %}

JSDoc wouldn't recognize `Urlify` as part of the `Absolventa` namespace, nor would it find
the `Absolventa.Urlify.init()` static method.

Behold the necessary JSDoc tags to tie it all together:

{% highlight javascript %}
/**
 * @namespace Absolventa
 */
(function() {
    window.Absolventa = window.Absolventa || {};
}());

/**
 * @namespace Urlify
 * @memberof Absolventa
 * @requires {@link Absolventa.Helpers}
 */
(function() {
    "use strict";
    Absolventa.Urlify = (function() {
        var init;

        /**
         * @function init
         * @memberof! Absolventa.Urlify
         * @param {string} foo
         * @example
         * Absolventa.Urlify.init('hello world')
         */
        init = function(foo) {
          // Magick!
        };

        return {
          init : init
        };
    }());
}());
{% endhighlight %}

Et voilá! All inner functions must be declared with a `@memberof!` and a
`@function <name>` tag. Note that it defines namespaces where we would refer to
it as modules, but it's just for the sake of documentation … and it's a
namespace after all.
