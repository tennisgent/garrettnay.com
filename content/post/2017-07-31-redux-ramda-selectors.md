+++
title = "Better Redux Selectors with Ramda"
date = "2017-07-31T09:18:32-06:00"
description = "Ramda makes writing Redux selectors easier and, dare I say, more declarative."
+++

I love Redux. Big deal, right? [A lot of people do](https://github.com/reactjs/redux/stargazers). But seriously, Redux provides a great system for managing state in complex JavaScript applications. Not only that, but it was also the first Flux-like library that I really felt I understood.

I also love [Ramda](http://ramdajs.com/). [I'm definitely not the only one there either](https://github.com/ramda/ramda/stargazers), although it's not quite on the same level of popularity as Redux. That's a shame, because the two libraries can work really well together. Although they serve two very different purposes, their ultimate goal is the same, which is to help you write more functional and declarative™ JavaScript applications.

In this article I want to show one area where Ramda can improve your Redux app: selector functions.

## Recap of Selector Functions

If you're not familiar with selector functions in Redux, here's the basic idea: Instead of having your view components reach directly into the state tree, you define functions that get that data for you. You can think of them as the public read API for your state. They aren't strictly necessary for using Redux—they're more convention than requirement. In the simplest applications you might not need them (in the simplest applications [you might not need Redux at all](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)), but they are useful if your state has a complex structure or if you need to compute some kind of derived data from it.

Selector functions take the Redux state tree object and return whatever data you need from it. Here are some examples.

- Getting a property off the state tree:

    ```js
    function getUserName(state) {
        return state.user.name
    }
    ```

    This example might seem a little silly, but the point is it hides the internal structure of the state from consuming components. Components don't care about the shape of the state; all they care about is getting the data they need.

- Deriving data from a property:

    ```js
    function isLoggedIn(state) {
        return state.user.id != null
    }
    ```

    Similar to the first example, only now instead of reading a property directly, we're calculating a value based on it—in this case, whether it's set or not. Calculating boolean values based on state properties are a common use case for Redux selectors. Other types of calculations are certainly possible too.

- Deriving data from a list of items:

    ```js
    function getTotalItemCount(state) {
        return Object.keys(state.items.byId)
            .reduce(function(total, id) {
                return total + state.items.byId[id].count
            }, 0)
    }
    ```

    Assuming each item object has a `count` property, this selector uses [`array.reduce`](https://devdocs.io/javascript/global_objects/array/reduce) to sum all the counts together. Note that the items are stored in an object called `byId`, where each key of the object is an ID and the value is the item itself, as opposed to storing them in an array. This organization technique is sometimes called state normalization and is recommended [in the Redux docs](http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.html) as well as [by Dan Abramov](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape), the original creator of Redux. Organizing the data in this way (paired with an `ids` array so that you can preserve order) allows for simpler lookups and updates of individual items.

    The drawback with this approach is that we need to go through some gymnastics if we ever need to access the data as an array as we do in this example. Since a plain object doesn't have a `reduce` method, we first call [`Object.keys`](https://devdocs.io/javascript/global_objects/object/keys) on the object to get an array of IDs and operate on that instead. It's a little ugly, but it works.

These examples are fairly generic, but they demonstrate the kinds of selectors you might find yourself writing in a Redux application. Let's see them all together, this time exporting them so they can be used in our components.

```js
export function getUserName(state) {
    return state.user.name
}

export function isLoggedIn(state) {
    return state.user.id != null
}

function getTotalItemCount(state) {
    return Object.keys(state.items.byId)
        .reduce(function(total, id) {
            return total + state.items.byId[id].count
        }, 0)
}
```

If you're in an environment that supports [ES2015 arrow functions](https://devdocs.io/javascript/functions/arrow_functions) (or you're using a transpiler like [Babel](https://babeljs.io/) that makes them available anywhere), you can make these selectors even nicer:

```js
export const getUserName = state => state.user.name

export const isLoggedIn = state => state.user.id != null

export const getTotalItemCount = state =>
    Object.values(state.items.byId)
        .reduce((total, item) => total + item.count, 0)
```

Those implicit returns make the functions nice and succinct. Not bad! Note that I replaced the `Object.keys` call with [`Object.values`](https://devdocs.io/javascript/global_objects/object/values), which, as the name implies, returns an array of the property *values* rather than the *keys*, making it less cumbersome to get the data we actually need. (`Object.values` is an ES2017 feature, and you'll need a polyfill to use it in environments that don't support it. Using Babel transforms isn't enough because it's a built-in method—you'll need something like [babel-poylfill](http://babeljs.io/docs/usage/polyfill/).)

We might be pretty happy with our selector functions at this point, especially with the concise ES2015 syntax. But can we make them better? Let's take a look at Ramda and see what it can do for us.

## What's Important to Understand about Ramda

Ramda bills itself as "A practical functional library for JavaScript programmers." Your reaction to that description might be one of the following:

1. Cool!
2. I already have Lodash for that.
3. I've got my arrow functions and array methods and I don't need anything else, thank you very much.

Before I try to persuade you to join the #1 camp, let me start off with a disclaimer. You don't *need* Ramda, any more than you need React to build view components. Bringing in a third-party library is a decision you should weigh carefully, considering costs such as increases to the size of your application, time for new team members to get up to speed, etc. And it's true that you can accomplish anything Ramda does with vanilla JavaScript. But like any third-party library, Ramda can help you stop worrying about mundane implementation details and focus instead on the unique logic of your application. That's my response to reaction #3 above.

Now let's talk about #2. Why should you choose Ramda over Lodash? Lodash is a great library, and I've used it successfully in several projects. But let's take a look at what makes Ramda different, and for that I'll quote the [Ramda homepage](http://ramdajs.com/):

> The primary distinguishing features of Ramda are:
>
> - Ramda emphasizes a purer functional style. Immutability and side-effect free functions are at the heart of its design philosophy. This can help you get the job done with simple, elegant code.
> - Ramda functions are automatically curried. This allows you to easily build up new functions from old ones simply by not supplying the final parameters.
> - The parameters to Ramda functions are arranged to make it convenient for currying. The data to be operated on is generally supplied last.

Those last two points are what I want to emphasize when dealing with Redux selectors.

### Currying

Ramda functions are automatically curried. What is [currying](https://en.wikipedia.org/wiki/Currying)? It sounds delicious! Basically, currying is the idea of transforming a function that takes N arguments into a sequence of N functions that take one argument each.

Let's look at the quintessential example, add. A basic, non-curried adding function might look like this:

```js
function add(a, b) {
    return a + b
}

//or
const add = (a, b) => a + b
```

But the curried form of add would look like this:

```js
const addCurried = a => b => a + b
```

Notice the difference? Now instead of taking both arguments at once, `addCurried` first takes one argument and returns another function that takes the second argument. Arrow functions make currying much more convenient to write in JavaScript than it used to be.

We can use curried functions to create reusable functions with partially applied arguments:

```js
// create a function that adds 2 to its argument
const add2 = addCurried(2)

add2(5) // 7
add2(13) // 15
```

As it turns out, Ramda does a sort of magic currying (yum!) that allows its functions to be used in both curried and non-curried form. In other words, if you have all the arguments for a function at once, you can call it like a regular function (e.g. `add(1, 2)`), instead of always having to use the curried form (e.g. `add(1)(2)`), which is a little unwieldy when you have all the arguments.

In case you're wondering, [Ramda does have an `add` function](https://devdocs.io/ramda/index#add).

### Data Last

Adding is the "Hello world" of currying examples. Let's look at something a little more complex to see why it's helpful that Ramda functions take data as the last parameter.

Let's say we have an array of numbers, and we want multiply each number by 2. With a curried `multiply` function we can easily create an `double` function that doubles whatever number you pass to it.

*Note: from now on in this article, all Ramda functions will be prefixed with an `R` so it's clear where they come from. It's also possible to import specific functions. If you do that, and you're using Babel, I recommend checking out the [Ramda Babel plugin](https://github.com/megawac/babel-plugin-ramda) to help keep your builds smaller.*

```js
// create a function that always multiplies by 2
const double = R.multiply(2)
```

Now how do you apply a function to every element in array to get a new array? by using a `map`, right?

```js
const nums = [1, 2, 3, 4]

nums.map(double) // [2, 4, 6, 8]
```

Not bad. By passing in a named function to `map` that describes the transformation taking place, it's easier to see what's going on. But although the `double` function is reusable, the `map` isn't. You need to call `map` on every array you want to apply this transformation to.

Ramda has a `map` function as well, but it has a significant difference from the built-in `map` function as well as Lodash's `map` function: you pass the transformation function first, followed by the array to be operated on.

```js
const nums2 = [11, 12, 13, 14]

R.map(double, nums2) // [22, 24, 26, 28]
```

So far we haven't gained anything, but remember that Ramda functions are automatically curried. That means we can pass the first argument to `map` and get a new function that takes the second argument.

```js
// create a new function that doubles every element of an array
const doubleAll = R.map(double)

doubleAll(nums) // [2, 4, 6, 8]
```

See what we just did there? We created a new function that takes an array and returns a new one with each element doubled. And the reason we could do that is that the `map` function takes the array—the data to be operated on—last. The data is the part most likely to change, so by putting it last, we can much more easily build up expressive functions that can be reused with multiple pieces of data. When the data is supplied first, as in the built-in `map` function and Lodash's `map`, we're more likely to write a bunch of one-off functions that are less expressive, because the data is always changing.

I hope you now see how these two attributes—automatically curried functions and taking data last—make Ramda more than a simple utility library. It's a toolkit for writing more declarative, functional code.

Now let's see how Ramda helps us write better Redux selectors.

## Writing Selectors with Ramda

Let's go back to the selector examples we wrote earlier and rewrite them using Ramda.

### `getUserName`

For this one we need to get nested properties off the `state` object. Ramda happens to have a convenient function for this purpose, [`path`](https://devdocs.io/ramda/index#path). The first parameter is an array defining the path, and the second is the object to find the property on.

Let's use an arrow function for brevity's sake.

```js
export const getUserName = state => R.path(['user', 'name'], state)
```

One nice benefit of using `path` is that if any part of the path is undefined, it will return `undefined` without throwing an error. Granted, if you've set up your Redux store properly, the `user` object should never be undefined, but it's still a nice way to avoid that dreaded error.

Remember how Ramda functions are curried? Let's use the curried form of `path` instead:

```js
export const getUserName = state => R.path(['user', 'name'])(state)
```

Wait a second. Do you see something odd here? We've now defined a function that takes an object `state` and creates a new function that takes that same object `state`. In the end we've written a function that calls another function with exactly the same arguments. That sounds to me like an unnecessary layer, so let's remove it!

```js
export const getUserName = R.path(['user', 'name'])
```

Boom. There you have it. [Try it out](http://ramdajs.com/repl/#?const state %3D {%0A user%3A {%0A name%3A 'Johann'%0A }%0A}%0A%0A%2F%2F const getUserName %3D state %3D> R.path(['user'%2C 'name']%2C state\)%0Aconst getUserName %3D R.path(['user'%2C 'name']\)%0A%0AgetUserName(state\)); you'll see that it works exactly the same way.

Notice that we've now defined a function without ever mentioning the data it operates on. This technique is what is known as [point-free style or tacit programming](https://en.wikipedia.org/wiki/Tacit_programming). The idea of operating on data without ever identifying the arguments that are being operated on may sound bizarre—it certainly did to me when I first learned about it. I'll discuss some of the benefits of point-free style at the end of the article. For now, at the very least we can see that it makes for a more concise function definition, and who doesn't love cutting down on typing?

Ramda's automatic currying and order of parameters are what make point-free style possible.

### `isLoggedIn`

Our next selector, `isLoggedIn`, is similar to the first one except it calculates a boolean based on a state property. Here's the original, non-vanilla version for reference:

```js
export const isLoggedIn = state => state.user.id != null
```

It's a simple enough function, but could we make it more, well, functional? In other words, could we rewrite it in such a way that it is built up from smaller, potentially more reusable functions?

With Ramda we certainly can!

A way I've found helpful for rewriting functions like this is to ask myself: What am I trying to learn from the data? In this case, how do I define being logged in based on the state? According to this function, the user is logged in when the path `user.id` on the state object is not null or undefined. Therefore, if we could define a function `pathIsNotNullOrUndefined` to which we could pass the path and the state object, we'd be well on our way to Functional Town!

So what we're looking for is a function we could call like this:

```js
export const isLoggedIn = pathIsNotNullOrUndefined(['user', 'id'])
```

Based on the name of the function, we can see there are two parts: finding the path and determining if a value is not null or undefined. Sounds like a place for making some more functions. We already know we have `path`, so let's take a crack at the first part, and then we'll se how we can put the two together.

We could define our own `isNotNullOrUndefined` (or `isSet`, or whatever you'd like to call it) without too much difficulty, but for the sake of this article let's look at what Ramda has to offer us.

Sometimes it can be hard to find the right Ramda function because their names are based on functional languages like Haskell and are less common in the JavaScript world. But it turns out Ramda [has a function called `isNil`](https://devdocs.io/ramda/index#isNil) that returns `true` if the input value is `null` or `undefined`.

```js
R.isNil(null) // true
R.isNil(undefined) // true
R.isNil(false) // false
```

Great! Now all we need to do is negate it. We could do it like this:

```js
const isNotNil = val => !R.isNil(val)
```

Or we could go even further with using Ramda functions. You guessed it! There is a [`not` function](https://devdocs.io/ramda/index#not) that does basically the same thing as `!`:

```js
const isNotNil = val = R.not(R.isNil(val))
```

The second version seems silly, but now we're using nothing but functions. Yay! But we can do better. Now that you've had a taste of point-free style, you'll hopefully start looking for opportunities to refactor to that style. Can we define a function that doesn't reference `val` at all?

Yes. Yes, we can. But instead of using `not`, we'll use [a different function called `complement`](https://devdocs.io/ramda/index#complement). Whereas `not` immediately returns the complement of the *value* you pass to it (what you would get by putting a `!` in front of it), `complement` takes a *function* and returns a new function that, when called, returns the complement of whatever the original function returns.

If that doesn't make sense, seeing it in action might clear it up:

```js
const isNotNil = R.complement(R.isNil)

isNotNil(true) // true
isNotNil(null) // false
isNotNil(undefined) // false
```

Great, so now we have an `isNotNil` function, written in that sweet, sweet point-free style for good measure.

Now how do we put that together with `path`? First let's try to the naive approach:

```js
const pathIsNotNil = (path, state) => isNotNil(R.path(path, state))
```

Now remember, up above we said we wanted the function `pathIsNotNil` to take a path argument and return a new function that takes the state object so we can use it in a point-free way—in other words, we want it to be curried. Thanks to arrow functions, we can do that without changing too much:

```js
const pathIsNotNil = path => state => isNotNil(R.path(path, state))
```

Now it's curried. Beautiful. But can we get rid of that pesky `state` parameter and make it even more beautiful?

Ramda's [`compose` function](https://devdocs.io/ramda/index#compose) can help us here. The concept of [function composition](https://en.wikipedia.org/wiki/Function_composition_\(computer_science\)) is not unique to Ramda; it's a fundamental part of functional programming and has its roots in mathematics. (Side note: the deeper you get into functional programming, the more it will feel like you are doing math. I suppose that's because you're using functions in the truest sense of the word, just like functions in algebra. See? All those years of math *did* turn out to be useful!)

If you've ever used Lodash's `flow` function, you've used function composition, although in Ramda the functions are invoked from right to left rather than left to right, which is typical most functional languages. For example, using `compose` we can rewrite the above version of `pathIsNotNil` like so:

```js
const pathIsNotNil = path => state => R.compose(isNotNil, R.path)(path, state)
```

In this composed function, we pass the arguments `path` and `state` to the rightmost function, `R.path` first. The return value of that function is then passed to the next-rightmost function—in this case `isNotNil`—and so on until we come to the end of the functions and get a final return value. One important thing to remember here is that the rightmost function can take any number of arguments, but the rest of them have to take only one.

If the right-to-left composition seems weird to you, think of it this way: if the functions were nested inside each other as we originally had them, which function would be evaluated first? The innermost one, or in other words, the rightmost one. Alternatively, you can use Ramda's [`pipe` function](https://devdocs.io/ramda/index#pipe), which is the left-to-right version of `compose`, but I encourage you to try to get used to right-to-left composition because that is the way it is typically done.

Now can we get rid of the `state` parameter? Given what we know about how Ramda functions work, we might try this:

```js
// This doesn't actually work 😦
const pathIsNotNil = path => state => R.compose(isNotNil, R.path)(path)(state)
```

If we could do that, then we could take out the `state` parameter entirely. Unfortunately, composing doesn't work that way. According to [the Ramda docs](http://devdocs.io/ramda/index#compose), "The result of compose is not automatically curried." So we can't pass arguments to a composed function one at a time.

But all is not lost! The `path` function *is* curried, so we can partially apply that specific function with the path argument, resulting in a composed function that takes only one argument to begin with, like so:

```js
// This works! 😄
const pathIsNotNil = path => state => R.compose(isNotNil, R.path(path))(state)
```

With that in place, now we can remove `state`, resulting in our final, beautiful version of `pathIsNotNil`:

```js
const pathIsNotNil = path => R.compose(isNotNil, R.path(path))
```

You may be tempted to keep going and try to take the `path` parameter as well, making it 100% point-free. However, because of the limitations of `compose`, I'm not sure it's possible to do. I certainly haven't found a way to do it; if you have, let me know! In any case, the version we have here is pretty good. And now it's ready to be used in our selector:

```js
export const isLoggedIn = pathIsNotNil(['user', 'id'])
```

Hurrah! We did it! And what's more, now we have a couple utility functions (`isNotNil` and `pathIsNotNil`) that will almost certainly be useful elsewhere in the application.

[Try out this example in the Ramda REPL](http://ramdajs.com/repl/#?%2F%2F%20Note%3A%20All%20Ramda%20functions%20are%20automatically%20imported%20without%20the%20R%20namespace%0A%0Aconst%20state%20%3D%20%7B%20user%3A%20%7B%20id%3A%20%27abac%27%20%7D%20%7D%0A%0A%2F%2F%20Old%20version%0A%2F%2F%20export%20const%20isLoggedIn%20%3D%20state%20%3D%3E%20state.user.id%20%21%3D%20null%0A%0Aconst%20isNotNil%20%3D%20complement%28isNil%29%0A%0Aconst%20pathIsNotNil%20%3D%20p%20%3D%3E%20compose%28isNotNil%2C%20path%28p%29%29%0A%0Aconst%20isLoggedIn%20%3D%20pathIsNotNil%28%5B%27user%27%2C%20%27id%27%5D%29%0A%0AisLoggedIn%28state%29).

### `getTotalItemCount`

[next example](http://ramdajs.com/repl/#?const%20state%20%3D%20%7B%0A%20items%3A%20%7B%0A%20byId%3A%20%7B%0A%20abc123%3A%20%7B%20count%3A%202%20%7D%2C%0A%20abc125%3A%20%7B%20count%3A%204%20%7D%0A%20%7D%0A%20%7D%0A%7D%0A%0A%2F%2F%20function%20getTotalItemCount%28state%29%20%7B%0A%2F%2F%20return%20Object.keys%28state.items.byId%29%0A%2F%2F%20.reduce%28%28total%2C%20id%29%20%3D%3E%20total%20%2B%20state.items.byId%5Bid%5D.count%2C%200%29%0A%2F%2F%20%7D%0A%0Aconst%20sumCounts%20%3D%20compose%28sum%2C%20values%2C%20map%28prop%28%27count%27%29%29%29%0A%0Aconst%20pathOrObj%20%3D%20pathOr%28%7B%7D%29%0A%0Aconst%20getTotalItemCount%20%3D%20compose%28sumCounts%2C%20pathOrObj%28%5B%27items%27%2C%20%27byId%27%5D%29%29%0A%0A%2F%2F%20getTotalItemCount%28state%29%0A%0Areduce%28%28total%2C%20item%29%20%3D%3E%20total%20%2B%20item.count%2C%200%2C%20state.items.byId%29)

## Why?
