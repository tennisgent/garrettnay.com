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

- Getting a list of items:

    ```js
    function getItems(state) {
        return state.items.ids.map(function (id) {
            return ids[id]
        })
    }
    ```

    This selector assumes your items are stored by two different reducers: one that keeps a list of IDs and one that keeps them in an object keyed by ID. This normalization of state structure is recommended [in the Redux docs](http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.html). Structuring the data this way makes it easier to update individual objects (by looking them up in the key-value object) while still preserving a specific order (by tracking their ids in an array). To get the full objects as an array, though, we need to map over the ids like this, which is why a selector function is helpful.

- Deriving data from a list of items:

    ```js
    function getTotalItemCount(state) {
        return getItems(state)
            .reduce(function (total, item) {
                return total + item.count
            }, 0)
    }
    ```

    Assuming each item object has a `count` property, this selector uses [`array.reduce`](https://devdocs.io/javascript/global_objects/array/reduce) to sum all the counts together. Note that since it needs the list of items to operate on, it calls the `getItems` selector to get that list. As you write more complex selectors that derive data, you'll find that using other selectors helps to simplify things.

These examples are fairly generic, but they demonstrate the kinds of selectors you might find yourself writing in a Redux application. Let's see them all together, this time exporting them so they can be used in our components.

```js
export function getUserName(state) {
    return state.user.name
}

export function isLoggedIn(state) {
    return state.user.id != null
}

function getItems(state) {
    return state.items.ids.map(function (id) {
        return ids[id]
    })
}

function getTotalItemCount(state) {
    return getItems(state)
        .reduce(function (total, item) {
            return total + item.count
        }, 0)
}
```

If you're in an environment that supports [ES2015 arrow functions](https://devdocs.io/javascript/functions/arrow_functions) (or you're using a transpiler like [Babel](https://babeljs.io/) that makes them available anywhere), you can make these selectors even nicer:

```js
export const getUserName = state => state.user.name

export const isLoggedIn = state => state.user.id != null

export const getItems = state =>
    return state.items.ids.map(id => state.items.byId[id])

export const getTotalItemCount = state =>
    getItems(state)
        .reduce((total, item) => total + item.count, 0)
```

Those implicit returns make the functions nice and succinct. Not bad!

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

[next example](http://ramdajs.com/repl/#?const%20isLoggedInOld%20%3D%20state%20%3D%3E%20state.user.id%20%21%3D%20null%0A%0Aconst%20state%20%3D%20%7B%20user%3A%20%7B%20id%3A%20%27abac%27%20%7D%20%7D%0A%0A%2F%2Fconst%20isLoggedIn%20%3D%20state%20%3D%3E%20not%28isNil%28path%28%5B%27user%27%2C%20%27id%27%5D%2C%20state%29%29%29%0A%0Aconst%20isNotNil%20%3D%20complement%28isNil%29%0A%0Aconst%20pathIsNotNil%20%3D%20p%20%3D%3E%20compose%28isNotNil%2C%20path%28p%29%29%0A%0Aconst%20isLoggedIn%20%3D%20compose%28isNotNil%2C%20path%28%5B%27user%27%2C%20%27id%27%5D%29%29%0A%0Aconst%20isLoggedIn2%20%3D%20pathIsNotNil%28%5B%27user%27%2C%20%27id%27%5D%29%0A%0AisLoggedIn2%28state%29)

## Why?
