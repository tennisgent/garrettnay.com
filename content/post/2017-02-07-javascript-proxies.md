+++
date = "2017-02-07T21:47:58-07:00"
title = "Use JavaScript Proxies to Debug Changes to Objects"
description = "How to use a JavaScript proxy to track down where objects are being mutated."
+++

I recently found a nifty use for JavaScript proxies that helped me understand a complex application better. It's nothing earth-shattering, and I'm certainly not the first person to have this idea, but I thought I'd share it.

<!--more-->

## The State Problem

My particular situation was in an application written in Angular 1. I started a new job at the beginning of the year and was assigned to work on this app, which posed a significant learning curve for me—not just because I've never done any significant work with Angular before, but also because the app is full of complex state mutations that are difficult to track down. When a certain piece of state was mysteriously changing out from under me, I needed a way to find out what was changing it.

## Enter the Debug Proxy

The solution I came up with was to replace the mutating object, which was Angular service, with a proxy that logged every time a property changed. Here's the gist of it:

```js
// Function defining the service
function FileSaveService () {
  const service = {
    action: 'SAVE',
    isSaving: false,
    // ... other properties
  };

  return new Proxy(service, {
    set (target, propKey, value) {
      console.error(`Setting FileSaveService.${propKey}`, {
        from: target[propKey],
        to: value
      });
      return Reflect.set(target, propKey, value);
    }
  });
}
```

I won't go into a lot of detail about what proxies are or how they work (consult [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) or ["Metaprogramming with Proxies"](http://exploringjs.com/es6/ch_proxies.html) by Dr. Axel Rauschmayer for much more detail on the subject). The important thing to understand here is that instead of returning the actual `service` object itself, this code returns a proxy for the service, allowing us to intercept different ways the object is interacted with. The second argument to the `Proxy` constructor is a handler, and in it we define just one method, `set`. That tells the proxy object that whenever something tries to set a property on the object, it should call this `set` method instead, with information about what is being changed and what value it is being changed to.

In our case, we just log that information, both what the property currently is set to (`target[propKey]`) and what is being changed to (`value`). Why use `console.error`? Because it provides stack traces, allowing you to look up exactly where in your code the property is being set.

{{< figure src="/images/post/proxies/stacktrace.png" title="The stack trace shows you what function caused the mutation." alt="Screenshot of a console.error stack trace in a browser console.">}}

This example is admittedly trivial, since all the code is one place, but I'll tell you, when there are dozens of files and you don't know what half of them do, this proxy combined with `console.error` will go a long way toward helping you understand things.

By the way, I don't recommend doing this in production. Proxies have many uses, but this particular one is only good for when you're knee-deep in debugging work. Be sure to clean up after yourself and not deploy this kind of code to production.

## Moral of the Story

You may be thinking at this point, "But you shouldn't be mutating state like that in the first place." And I would wholeheartedly agree with you. If you need to drop in a proxy just to understand where a state mutation is taking place, that is a pretty strong code smell, and you should rethink how the state of your application is managed.

There is a reason libraries like [Redux](http://redux.js.org/) are so popular. Over the past couple of years many JavaScript developers, including me, have begun to learn how much easier it is to work in an application where all (or most) of the state is gathered into one place and all the changes to it are explicit. When you have those kinds of restrictions in place, data flow becomes more predictable, and it is much easier to track down bugs and add new features. After working with Redux for the better part last year, it was hard to come to a project that started in the era when two-way binding was still the new hotness and mutations were encouraged.

Alas, oftentimes our job as developers is working in legacy code. If your situation is similar to mine, you might find this trick with proxies helpful.
