+++
title = "Write Modular JavaScript That Works Anywhere with UMD"
date = "2014-02-19T06:20:47-07:00"
description = "How the Universal Module Definition helps you write JavaScript modules that work in multiple environments, including RequireJS and Node."
+++

Have you ever used a JavaScript library that gives you more than one way to include it in your project? Take, for example, the [Backbone localStorage adapter](https://github.com/jeromegn/Backbone.localStorage). The readme gives you two ways to put it into your project: including it through a separate script tag in your HTML, or loading it through [RequireJS](http://requirejs.org/).

As a developer that is still learning to navigate the JavaScript landscape, I've wondered how libraries like that do it. The two methods of inclusion work entirely differently. When you include a library using a script tag, you're usually creating global variables that other scripts can use. But one of the main benefits of using RequireJS is that it eliminates the need for global variables. How can the same resource be loaded in both ways?

After some digging, I learned that many of these projects use a pattern called the [Universal Module Definition](https://github.com/umdjs/umd).

<!--more-->

## Modularity in Javascript

FIrst, a word about modularity in Javascript. I'm a little late to this party, but the current wisdom in JavaScript development seems to be that your application code should be modular. I take "modular" to mean a few things:

*   Your application is split up into distinct, self-contained pieces (modules), each responsible for a specific bit of functionality.
*   A module doesn't add any unnecessary variables or functions to the global namespace. It is *encapsulated*---it exposes an interface for other modules to use, but that's it.
*   Modules are potentially reusable in other contexts.
*   Modules do not depend rigidly on each other. In other words, they're loosely coupled, and dependencies could probably be swapped without too much trouble.

Probably not the best definition in the world (Addy Osmani [explains it much better than I can](http://addyosmani.com/writing-modular-js/)), but it captures the basic idea as far as I understand it.

The problem with JavaScript is that for a long time, there wasn't a really good way to make code modular. When you're just putting in a bunch of script tags, the only way the scripts can use each other is if they create global variables, and global variables get messy pretty quickly. It's also hard to make sure dependencies are resolved correctly—all you can do is keep track of the order the scripts are loaded in, which gets tedious after a while. Limitations like these are the reason why modular JavaScript is a relatively new concept, while other programming languages have had it standard from the beginning.

As it turns out, built-in module functionality [is coming in the ECMAScript 6 specification](http://wiki.ecmascript.org/doku.php?id=harmony:modules), but that still appears to be a ways off. In the meantime, other, less-official specifications have emerged to fill in the gap. The big players are [CommonJS](http://www.commonjs.org/) and [Asynchronous Module Definition (AMD)](https://github.com/amdjs/amdjs-api/blob/master/AMD.md). CommonJS is the system used in [Node.js](https://nodejs.org/), and the most popular implementation of AMD is [RequireJS](http://requirejs.org/), which I like to use on the front end.[^1]

These specifications work quite differently. But suppose you're writing a library in JavaScript. This library could be useful both in the browser, where AMD is frequently used, or on the server, where CommonJS is commonly used (heh). On top of that, you also want to make this library accessible to developers who want to use it the old-fashioned way, via script tags. How do you do it?

There are many ways you could go about solving this problem. But one solution I particularly like is UMD—the Universal Module Definition.

## The Universal Module Definition

UMD is actually nothing more than a collection of patterns you can find in [a repository on GitHub](https://github.com/umdjs/umd). There are lots of different patterns, or definitions, each for a different set of requirements. But they're all essentially variations on the same idea.

For example, let's look at [returnExports.js](https://github.com/umdjs/umd/blob/master/templates/returnExports.js):

```javascript
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD. Register as an anonymous module.
        define(['b'], factory);
    } else if (typeof exports === 'object') {
        // Node. Does not work with strict CommonJS, but
        // only CommonJS-like environments that support module.exports,
        // like Node.
        module.exports = factory(require('b'));
    } else {
        // Browser globals (root is window)
        root.returnExports = factory(root.b);
    }
}(this, function (b) {
    //use b in some fashion.

    // Just return a value to define the module export.
    // This example returns an object, but the module
    // can return a function as the exported value.
    return {};
}));
```

This definition makes your module available as a CommonJS module, an AMD module, or a global variable, depending on the environment. How? The magic lies in one of those peculiar features of JavaScript, the [immediately-invoked function expression (IIFE)](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression).

I won't go into a lot of detail about how IIFEs work (see [Ben Alman's article](http://benalman.com/news/2010/11/immediately-invoked-function-expression/) for an excellent explanation). In a nutshell, an IIFE consists of the following things:

1. a function definition
2. parentheses wrapped around the function to turn it into a function *expression*, which can be executed immediately, whereas a function *declaration* cannot
3. a pair of parentheses at the end, optionally containing arguments to be passed into the function; these parentheses signal that the function is to be executed right away

IIFEs are a useful tool for encapsulating code, because variables declared inside the function are not accessible outside it. In the case of UMD, the IIFE gives you control over exactly how the module is exposed.

Let's go back to the `returnExports` example. Here it is again with the internals stripped out:

```javascript
(function (root, factory) {

    // environment detection here

}(this, function (b) {

    // module definition here

}));
```

The function has two parameters, `root`, and `factory`. If you look at the arguments at the bottom, you can see that the argument passed in for `root` is `this`, which represents the global object (the `window` object in a browser context). Although the global object is accessible inside the function (since it's global), using a local variable to reference it is often a good idea because it shortens the lookup chain and can possibly improve performance.

The `factory` parameter gets the value of a function that is passed in as the second argument. This factory function is where the module is actually defined. It's the equivalent of the `define` function in RequireJS, or the function you assign to `module.exports` in Node. As in those individual situations, this function can return anything, but usually it's going to be an object or a function.

Now let's look at what happens inside the main function:

```javascript
if (typeof define === 'function' && define.amd) {
    // AMD. Register as an anonymous module.
    define(['b'], factory);
} else if (typeof exports === 'object') {
    // Node. Does not work with strict CommonJS, but
    // only CommonJS-like environments that support module.exports,
    // like Node.
    module.exports = factory(require('b'));
} else {
    // Browser globals (root is window)
    root.returnExports = factory(root.b);
```

I call this part the environment detection. Here we have a series of if-then statements to determine what kind of module loader is present, if there is any. First we test if it's an AMD loader by looking for a `define` function; that function should also have a property called `amd`. If it's there, we use the `factory` function as the callback for `define`, thus defining an AMD module.

If an AMD loader isn't there there, we then look for the CommonJS `exports` object. As the comments in the code mention, this part doesn't follow the strict CommonJS specification, but there is [another definition for that](https://github.com/umdjs/umd/blob/master/templates/commonjsStrict.js). As with AMD, if we are using a CommonJS-like loader, then the function assigns the `factory` function to `module.exports`, making it ready to be loaded as a Node module.

If neither of those loaders are present, then the last option is to load the module as a global variable. This is where the `root` parameter comes in: the script assigns the `factory` function as a property of `root`, which in most cases is going to point to `window`. Here they have called the global variable `returnExports`, but that's just because that's the name of the module pattern we're using—you can call it anything. If the script gets to this point, the module will be available as a global, and so the script can be included through an HTML script tag, and other scripts will be able to use it.

You might have noticed an argument called `b` that's getting passed to the `factory` function. That's basically just a demonstration of how dependencies are handled in UMD. If your module depends on other modules, you can put them in the way `b` is done here. If not, you can take that part out. In fact, [the actual returnExports definition](https://github.com/umdjs/umd/blob/master/templates/returnExports.js) includes a variant that uses no dependencies, so you can follow that one if it applies.

## Using UMD

And there you have it! Now you have a way to write a JavaScript module that can be loaded by both of the major module-loading systems, or as a global when neither of those loaders are present. What I especially love about UMD is that the global is created *only* when the loaders aren't present, so if you're using RequireJS, you're not going to see this module as a global.

The definition we looked at here is only one of many included in the UMD repository. Be sure to check it out the other versions to see if there is a different ones that better fits the particular requirements of the module you're writing.

The downside of the UMD approach is that it does add a bit of boilerplate to your code. I personally don't think it's that much, but I suppose it could get old after a while. To make things a little easier, I created [a package of Sublime Text snippets](https://packagecontrol.io/packages/UMD%20snippets) for all the UMD patterns. If you use Sublime Text, that package should hopefully make it a little easier and quicker to incorporate UMD into your workflow. There is also [a Grunt plugin](https://github.com/quandora/grunt-umd-wrapper) and [a Gulp plugin](https://github.com/phated/gulp-wrap-umd) available to automate the process of wrapping your code in UMD.

So there you go. I hope this post has helped you understand how UMD works and encouraged you to start using it in your own projects.

[^1]: I am also aware of a project called [Browserify](http://browserify.org), which lets you load modules in Node fashion in the browser. It's really picking up steam lately, but my experience is mainly with AMD and RequireJS, so that's what I'm focusing on here. But since Browserify uses Node-style module loading, UMD can still be used for it.
