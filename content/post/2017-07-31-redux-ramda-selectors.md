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

    Assuming each item object has a `count` property, this selector uses [`array.reduce`](https://devdocs.io/javascript/global_objects/array/reduce) to sum all the counts together. Note that the items are stored in an object called `byId`, where each key of the object is an ID and the value is the item itself, as opposed to storing them in an array. This organizational technique is sometimes called state normalization and is recommended [in the Redux docs](http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.html) as well as [by Dan Abramov](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape), the original creator of Redux. Organizing the data in this way (paired with an `ids` array so that you can preserve order) allows for simpler lookups and updates of individual items.

    The drawback with this approach is that we need to go through some gymnastics if we ever need to access the data as an array as we do in this example. Since a plain object doesn't have a `reduce` method, we first call [`Object.keys`](https://devdocs.io/javascript/global_objects/object/keys) on the object to get an array of IDs and operate on that instead. It's a little ugly, but it works.

These examples are fairly generic, but they demonstrate the kinds of selectors you might find yourself writing in a Redux application. Let's see them all together, this time exporting them so they can be used in our components.

```js
export function getUserName(state) {
    return state.user.name
}

export function isLoggedIn(state) {
    return state.user.id != null
}

export function getTotalItemCount(state) {
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

We can use curried functions to create reusable functions with [partially applied](https://en.wikipedia.org/wiki/Partial_application) arguments:

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

*Note: from now on in this article, all Ramda functions will be prefixed with an `R` so it's clear where they come from, just like in the [Ramda documentation](https://devdocs.io/ramda/). It's also possible to import specific functions. If you do that, and you're using Babel, I recommend checking out the [Ramda Babel plugin](https://github.com/megawac/babel-plugin-ramda) to help keep your builds smaller.*

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

Now for our most complicated selector yet, `getTotalItemCount`. Instead of looking at a single state value, it operates on a list of them. Here's what we currently have:

```js
export const getTotalItemCount = state =>
    Object.values(state.items.byId)
        .reduce((total, item) => total + item.count, 0)
```

There's a fair amount of stuff going on here, and once you get to a certain level of complexity you'll find that there are multiple ways to break a problem down in a functional style. Because of that, I'm going to explain two different possibilities for how to accomplish it. They certailny aren't the only ways to do it, nor are they necessarily the best ways. Someone more advanced in FP and Ramda could almost certainly find a more efficient or sensible way to do it. I encourage you to try different ways and see if you can make it better.

As with the last example, let's frame our problem by considering what we are trying to learn from the data. Putting aside for a moment all the work we need to do to get to the relevant slice of the state tree, the meat of what we're trying to do is this: get the sum of every item's `count` property.

Let's create a function `sumCounts` that takes an array of items and returns the sum of each item's `count` property. Here's an initial stab at it using vanilla JavaScript:

```js
const sumCounts = items =>
    items.reduce((total, item) => total + item.count, 0)
```

Now we'd like to rewrite this using Ramda functions, ideally in a point-free style. Here's where branch into two different possibilities. One of them relies on using `map` to transform the items before summing the counts, and the other uses `reduce` to do it all at once.

#### Using `map`

Using `map` to transform the data into a form we need is probably the most obvious and straightforward way to do it. It's made even easier by the fact that Ramda has a [`sum` function](https://devdocs.io/ramda/index#sum) that takes an array of numbers and returns, well, the sum.

```js
const sumCounts = R.compose(R.sum, R.map(R.prop('count')))
```

Remember that `compose` passes our arguments (in this case, the array of items) to the rightmost function and then the return value of that function to the next one. The rightmost function here is a `map` that applies a function created by `prop('count')`.

We haven't seen [the `prop` function](https://devdocs.io/ramda/index#prop) before, but what it does is take the prop of the name you give it from the object you pass to it as the second argument. Since we're passing only the first argument here, it instead creates a function that takes the object to pull the prop from. So we can pass that partially applied function to `map`, which will pass each item in the array to it. The `map` function will therefore map each item to its `count` property, resulting in an array of numbers that can then be passed into `sum`.

As it turns out, using `map` in this way—to pluck a property off each object—is such a common usage that Ramda provides a convenience function for it called [`pluck`](https://devdocs.io/ramda/index#pluck). Using `pluck` makes our function even simpler:

```js
const sumCounts = R.compose(R.sum, R.pluck('count'))
```

There we go. It looks pretty good. Now we have a function to which we can pass an array of `items` that each have a `count` property and get the total count.

Before moving on to using this function with the actual state object, let's consider an alternative.

#### Using `reduce`

The mapping version of `sumCounts` that we created will get the job done. But I'd like to figure out how to do it with a `reduce` instead. If I didn't have Ramda at my disposal in a project and had to use vanilla JavaScript, I'd use `reduce`, as you can see above. Why? Because I like to avoid looping through an array more than once if I can. In the `map` version, we loop through the array once to map each item to its `count` and then once more to sum them up. That's not even counting the work we'll need to do before this to get to the array of items we need.

With `reduce`, we should be able to do both of those steps in one go. Perhaps this is a case of [premature optimization](https://en.wikipedia.org/wiki/Program_optimization#When_to_optimize), especially if it makes the code harder to understand (although I'll have more to say about unfamiliarity at the end of the article). It might even be a wash because of how many functions are being applied, which can reduce performance. But for argument's sake, and for the sake of better understanding some more Ramda concepts, let's work on a `reduce` version.

I have to admit that I really struggled with this part. But once I was able to frame the problem in generic terms, I was better able to find the functions I needed. You'll see what I mean shortly.

As a first step, we can use Ramda's [`reduce` function](https://devdocs.io/ramda/index#reduce), which, similar to the `map` function we used above, takes the reducing function first and the data second, allowing us to remove the `items` parameter completely.

```js
const sumCounts = R.reduce((total, item) => total + item.count, 0)
```

It's looking pretty good so far, but that anonymous function could use some improvement. Whenever I see a one-off anonymous function like that passed to functions like `map` or `reduce`, I like to think if there could be a way to use a named function that describes what's going on.

```js
const sumCounts = R.reduce(addCount, 0)
```

Doesn't that look better? Now we're moving closer to describing *what* we want from the data rather than *how* we're getting it. Of course, now we need to define an `addCount` function. Its plain extracted form looks like this:

```js
const addCount = (total, item) => total + item.count
```

Could we also define this function in terms of other functions? Let's see how Ramda can help us here. You probably thought you'd never use it, but let's use Ramda's `add` function instead of using addition syntax:

```js
const addCount = (total, item) => R.add(total, item.count)
```

All right, now here is where things start to get interesting. Notice how we've almost gotten to a point where we're passing the arguments directly along to another function. The only difference is that instead of passing `item` directly to `add`, we need to pull the `count` property off it. If we could find a way to transform that argument before it gets passed to `add`, we could do away with the arguments altogether and write this function in a point-free style. That may sound crazy, but stay with me.

We've already seen the `prop` function in the `map` example above. We can use that same function here:

```js
const addCount = (total, item) => R.add(total, prop('count', item))

// OR, using the curried form
const addCount = (total, item) => R.add(total, prop('count')(item))
```

Now that it's functions all the way down, let's take a step back and think about what we're doing in generic terms. We are taking two arguments and passing them to another function, leaving the first argument unchanged but applying a different function to the second before passing it through.

And you guessed it—Ramda has a function for that! It's called [`useWith`](https://devdocs.io/ramda/index#useWith), and it's one that I didn't understand the point of until I was working on this very problem. It will most likely look foreign to you if you're not used to programming in a functional way. It certainly did to me. I'll try to explain it.

`useWith` takes two parameters: a function and array of functions. The functions in the array are called transformer functions—they transform the argument that corresponds to their position in the array before passing it to the first function. In other words, the first function in the array transforms the first argument, the second function the second argument, and so on. The transformed arguments are then passed to the function that was passed as the first argument to `useWith`.

In our case here, the function we want the arguments to be passed to at the end is `add`, so we pass that as the first argument.

```js
const addCount = R.useWith(R.add, [/* transformers */])
```

We know that we need to transform the second argument of `add` by getting the `count` prop off it, so we can put that as the second function in the transformers array.

```js
const addCount = R.usseWith(R.add [/* 1st */, R.prop('count')])
```

What about the first argument? It corresponds to the total count. It's already a number, so we don't want to transform it at all. Here's where another seemingly useless function actually becomes useful. It's called [`identity`](https://devdocs.io/ramda/index#identity), and all it does is return the same value that was passed to it. It sounds silly by itself, but in a situation like this it is exactly what we need.

```js
const addCount = R.useWith(R.add, [R.identity, R.prop('count')])
```

Let's recap what we've created here. Using `useWith`, we wrapped the `add` function with some transformer functions. The first argument, which corresponds to the total count, is passed to the `identity` function and therefore remains unchanged. The second argument, which corresponds to an item in the array, is passed to the function created by `prop('count')` so we get the `count` property off it. Those two transformed arguments are then passed to `add`. The result is a function we can pass to `reduce`, just as we wanted:

```js
const addCount = R.useWith(R.add, [R.identity, R.prop('count')])

const sumCounts = R.reduce(addCount, 0)
```

If we want, we could make generic forms of these functions that work with any property name, in case we ever wanted to sum up different properties on a list of items.

```js
const addProp = propName => R.useWith(R.add, [R.identity, R.prop(propName)])

const sumProps = propName => R.reduce(addProp(propName), 0)

const sumCounts = sumProps('count')
```

We could take this even further by accepting a transforming function instead of hardcoding our own. For example, suppose each item had an array of comment IDs on it, and we wanted to sum the total length of each item's array of comments.

```js
const addTransformedItem = transformer =>
    R.useWith(R.add, [R.identity, transformer])

const sumTransformedItems = transformer =>
    R.reduce(addTransformedItem(transformer), 0)

const totalItemComments = R.compose(R.length, R.prop('comments'))

const sumComments = sumTransformedItems(totalItemComments)
```

Those probably aren't the best names, but naming things is hard. And whether or not it would be useful to abstract things this far depends on your data and how you're using it. Whichever avenue we choose to go down, we've successfully created a `sumCounts` function that's built using smaller, more expressive, reusable functions.

---

Now that we have our `sumCounts` function, we need to make it work with the shape of our data. As discussed above, the state object looks like this:

```js
const state = {
    items: {
        byId: {
            'item1': { id: 'item1', count: 2 },
            'item2': { id: 'item2', count: 4 },
            'item3': { id: 'item3', count: 7 }
        }
    }
}
```

The items aren't kept in an array, but `sumCounts` needs them to be an array. Ramda can help us here again with its [`values` function](https://devdocs.io/ramda/index#values), which is similar to `Object.values` except you don't need to worry about browser support.

We already know a function to get to a certain path on an object, we know a function to transform an object into an array, and we now have a function for summing up the counts. With those functions in hand, it's a matter of composing them together.

```js
export const getTotalItemCount =
    R.compose(sumCounts, R.values, R.path(['items', 'byId']))
```

[Try it out in the Ramda REPL](http://ramdajs.com/repl/?v=0.24.1#?const%20state%20%3D%20%7B%0A%20items%3A%20%7B%0A%20byId%3A%20%7B%0A%20%27item1%27%3A%20%7B%20id%3A%20%27item1%27%2C%20count%3A%202%20%7D%2C%0A%20%27item2%27%3A%20%7B%20id%3A%20%27item2%27%2C%20count%3A%204%20%7D%2C%0A%20%27item3%27%3A%20%7B%20id%3A%20%27item3%27%2C%20count%3A%207%20%7D%0A%20%7D%0A%20%7D%0A%7D%0A%0A%2F%2F%20map%20version%0A%2F%2F%20const%20sumCounts%20%3D%20R.compose%28R.sum%2C%20R.pluck%28%27count%27%29%29%0A%0Aconst%20addProp%20%3D%20propName%20%3D%3E%20R.useWith%28R.add%2C%20%5BR.identity%2C%20R.prop%28propName%29%5D%29%0A%0Aconst%20sumProps%20%3D%20propName%20%3D%3E%20R.reduce%28addProp%28propName%29%2C%200%29%0A%0Aconst%20sumCounts%20%3D%20sumProps%28%27count%27%29%0A%0Aconst%20getTotalItemCount%20%3D%0A%20R.compose%28sumCounts%2C%20R.values%2C%20R.path%28%5B%27items%27%2C%20%27byId%27%5D%29%29%0A%0AgetTotalItemCount%28state%29). Play around with it and try the different versions we talked about. See if you can make it even better.

## Wrapping Up: What We've Done

Whew! It's been a long journey, but we made it, successfully rewriting our Redux selectors in a functional, composable, point-free way. Let's recap everything we've built, putting it all together in one module.

```js
import R from 'ramda'

// Helper functions
const isNotNil = R.complement(R.isNil)
const pathIsNotNil = path => R.compose(isNotNil, R.path(path))
const addProp = propName => R.useWith(R.add, [R.identity, R.prop(propName)])
const sumProps = propName => R.reduce(addProp(propName), 0)
const sumCounts = sumProps('count')

// Selector functions
export const getUserName = R.path(['user', 'name'])
export const isLoggedIn = pathIsNotNil(['user', 'id'])
export const getTotalItemCount =
    R.compose(sumCounts, R.values, R.path(['items', 'byId']))
```

As you can see, not only have we rewritten our selectors in a more functional manner, we've also created a handful of helper functions as building blocks for the selectors. Those helper functions will almost certainly be useful elsewhere in the project code, whereas before everything was made with one-off anonymous functions that couldn't be reused. This is one area where Ramda really shines: it gives you a slew of useful functions to begin with, but the automatic currying and argument order make it easy for you to take those functions and make even more useful functions for your app's specific needs—functions that can be used in multiple places. I hope you're starting to become convinced of Ramda's usefulness.

## Why Did We Do This?

You might still be questioning whether we did in fact make the code any better by refactoring it this way. It's not like the original selector functions were difficult to understand. They were simple, pure functions to begin with, and that's great. Now we've added a layer of indirection by using a library, and the point-free style can look rather strange to the uninitiated.

Throughout this article I've talked about point-free style as though it's a goal in itself. If you've made it this far in the article, thank you for giving me the benefit of the doubt. Now let's talk about why point-free style can be a good thing for your code.

I'll start by saying that in other languages, point-free functions are entirely standard. In Haskell, for example, most functions are automatically curried (which is appropriate, since both Haskell and currying are named after the mathematician [Haskell Curry](https://en.wikipedia.org/wiki/Haskell_Curry)), so point-free style follows naturally from that. I know that "other people are doing it" in itself is a bad argument. I bring this up only to point out that Ramda isn't inventing some shiny new hotness here but is drawing upon decades of research and theory. It may be shiny and hot, but it's not new.

So why is it so popular outside of JavaScript Land? I admit I've sometimes had a hard time articulating the benefits, other than it's fun! But [this article from Randy Coulman](http://randycoulman.com/blog/2016/06/21/thinking-in-ramda-pointfree-style/) really helped me solidify things more. I've mentioned that point-free style can make your functions more concise. Great. But the real selling point for me is that it transforms the way you think about functions. Instead of focusing on the data itself, you start thinking more in terms of the transformations that need to take place to get what you need. It's less about the *how* and more about the *what*. It truly is more declarative. I know the "word" declarative gets thrown around a lot these days, but to me, point-free functions really are declarative because you're declaring relationships between data.

After all, what is functional programming? It's programming with functions, yes, but the best description I've heard as to what sets it apart from other styles of programming is this: the data is separate from the functions that operate on it. In essence, functional programming is about using functions to describe relationships between sets of data. You see some of that when using React, for example. A React component is essentially a function that turns application data into data that can be rendered.

When you write functions in a point-free style, you really make it about those relationships between sets of data. Now, as we saw with some of our functions up above, point-free style isn't always possible. Nor is it always desirable. But as you start to get into the habit of looking for opportunities to refactor to point-free, as Ramda makes possible, I hope you find that it changes your approach to functions in a beneficial way.

One final note in conclusion. Delving into functional programming territory can be uncomfortable. It certainly has been for me at times. As you change your project code like the examples we've gone through here, you may find co-workers (or even yourself) complaining that it makes the code more complex or harder to understand. To that I say this: Don't confuse simplicity with familiarity. It's a cliché thing to say at this point, but it's true. Many of us find FP difficult or complex because we were originally trained in a very different programming style. I believe that if a programmer was trained in FP from the very beginning, they would find object-oriented programming to be strange and unnecessarily complex.

So be aware of your biases when judging the complexity of a certain style of programming. That's not to dismiss the difficulty of adopting a different style, because it certainly does take some effort. But consider the long-term benefits against the short-term costs. I do believe that programming in a functional style, such as what we've done here, can lead to increased maintainability in your code, and the cost of maintenance always outweighs the cost of implementation.
