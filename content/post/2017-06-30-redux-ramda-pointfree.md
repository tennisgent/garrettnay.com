+++
date = "2017-06-30T08:00:00-06:00"
title = "Redux, Ramda, and Pointfree Selectors"
description = "Ramda helps you organize your Redux selectors in a clean and sensible way."
+++

I love Redux. Big deal, right? [A lot of people do](https://github.com/reactjs/redux/stargazers). But seriously, Redux provides a great system for managing state in complex JavaScript applications. Not only that, but it was also the first Flux-like library that I really felt I understood.

Although Redux is not directly tied to any particular view library, you'll typically find it in React applications. And much like React, the Redux library itself doesn't prescribe a specific way of organizing your app (aside from a few basic requirements). You're free to put your reducers, action creators, selectors, middleware, etc., wherever they make the most sense for you and your app (you don't even absolutely need any of those things except reducers). That's a good thing, because every app has different needs.

But that's not to say there aren't some best practices you could benefit from learning and applying. One such practice is something I initially ignored because I didn't understand its value: putting selector functions in the same module alongside their related reducer.

If you're not familiar with selector functions in Redux, here's the basic idea: Instead of having your view components reach directly into the state tree, you define functions that get that data for you. You can think of them as the public API for your state. They aren't strictly necessary for using Redux. In the simplest applications you might not need them (in the simplest applications [you might not need Redux at all](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)), but they are useful if your state has a complex structure or if you need to compute some kind of derived data from it.

Dan Abramov, in [one of the videos](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers) from his [course on idiomatic Redux](https://egghead.io/courses/building-react-applications-with-idiomatic-redux) (highly recommended),  brings up the idea of writing the selector functions in the same file where their related reducer is. For example, let's say you have a reducer for a section of the state tree called `user`. You might have a couple selectors for users for getting the user's name and checking if the user is logged in. If you have the reducer and selectors in the same file, it might look something like this:

```js
// define the reducer as the default export
export default function user(state = {}, action) {
  switch (action.type) {
    case 'LOGIN_FULFILLED':
      return action.user
    case 'LOGOUT_FULFILLED':
      return {}
    default:
      return state
  }
}

// all selector functions are named exports
export function getUserName(state) {
  return state.user.name
}

export function isLoggedIn(state) {
  return state.user.id != null
}
```

(If you see a problem with this, you're right. Stay with me.)

I saw this pattern when I was getting started with Redux, but I was resistant to the idea at first. It wasn't until I watched Dan Abramov's video that I understood the benefit of structuring things this way. What is the benefit? By colocating your selectors with your reducer, you are encapsulating the internal structure of that slice of the state tree. The only things that need to know about how the subtree is actually structured are right there in that file, and so any future changes to that structure would only affect that one file.

Let's say your backend developers decide they need to change the shape of the user object. That never happens, right? 
