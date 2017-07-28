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

- Getting a list of items:

    ```js
    function getItems(state) {
        return state.items.ids.map(function (id) {
            return ids[id]
        })
    }
    ```

    This selector assumes your items are stored by two different reducers: one that keeps a list of IDs and one that keeps them in an object keyed by ID. This normalization of state structure is recommended [in the Redux docs](http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.html).

- Deriving data from a property:

    ```js
    function isLoggedIn(state) {
        return state.user.id != null
    }
    ```

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

function getItems(state) {
    return state.items.ids.map(function (id) {
        return ids[id]
    })
}

export function isLoggedIn(state) {
    return state.user.id != null
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

export const getItems = state =>
    return state.items.ids.map(id => state.items.byId[id])

export const isLoggedIn = state => state.user.id != null

export const getTotalItemCount = state =>
    getItems(state)
        .reduce((total, item) => total + item.count, 0)
```

Those implicit returns make the functions nice and succinct. Not bad!

We might be pretty happy with our selector functions at this point, especially with the concise ES2015 syntax. But can we make them better? Let's take a look at Ramda and see what it can do for us.

## What's Important to Understand about Ramda

When you hear Ramda described as a functional JavaScript utility library, you may be tempted to think it's just a Lodash clone. But let me tell you, it is much more than that.

## Writing Selectors with Ramda

## Why?
