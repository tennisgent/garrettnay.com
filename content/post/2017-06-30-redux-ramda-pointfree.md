+++
date = "2017-06-30T08:00:00-06:00"
title = "Redux, Ramda, and Pointfree Selectors"
description = "Ramda helps you organize your Redux selectors in a clean and sensible way."
+++

I love Redux. Big deal, right? [A lot of people do](https://github.com/reactjs/redux/stargazers). But seriously, Redux provides a great system for managing state in complex JavaScript applications. Not only that, but it was also the first Flux-like library that I really felt I understood.

Although Redux is not directly tied to any particular view library, you'll typically find it in React applications. And much like React, the Redux library itself doesn't prescribe a specific way of organizing your app (aside from a few basic requirements). You're free to put your reducers, action creators, selectors, middleware, etc., wherever they make the most sense for you and your app (you don't even absolutely need any of those things except reducers). That's a good thing, because every app has different needs.

But that's not to say there aren't some best practices you could benefit from learning and applying. One such practice is something I initially ignored because I didn't understand its value: putting selector functions in the same module (i.e. file) alongside their related reducer.

If you're not familiar with selector functions in Redux, here's the basic idea: Instead of having your view components reach directly into the state tree, you define functions that get that data for you. You can think of them as the public read API for your state. They aren't strictly necessary for using Redux. In the simplest applications you might not need them (in the simplest applications [you might not need Redux at all](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)), but they are useful if your state has a complex structure or if you need to compute some kind of derived data from it.

## The Old Way: Separation by Function

In my early days of using Redux, I would structure my store something like the following. Let's say we have a reducer for user data and a couple of actions for logging in and out:

```js
// in src/store/reducers/user.js
export default function user(state = {}, action) {
  switch (action.type) {
    case 'LOGIN_FULFILLED':
      return { ...state, id: action.id }
    case 'LOGOUT_FULFILLED':
      return {}
    default:
      return state
  }
}
```

Then we might have some selector functions to help the view components get the data they need:

```js
// in src/store/selectors/user.js
export function getUserName(state) {
  return state.user.name
}

export function isLoggedIn(state) {
  return state.user.id != null
}
```

We might use this data, say, for a component up in the top bar of the application that displays different things depending on whether the user is logged in or not. Now that we have our selector functions, all we need to do is call them in our [`mapStateToProps` function](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options), passing them the entire state tree. We know we'll get the data from the state that we need.

```js
// in src/components/LogInOut.js
import { connect } from 'react-redux'
import { isLoggedIn, getUserName } from '../store/selectors/user'

function LogInOut({ isLoggedIn, name }) {
  return isLoggedIn
    ? <LogOut name={name}/>
    : <LogIn />
}

function mapDispatchToProps(state) {
  return {
    isLoggedIn: isLoggedIn(state),
    name: getUserName(state)
  }
}

export default connect(mapDispatchToProps)(LogInOut)
```
For a while, this convention of separating modules by their function (i.e. reducers, selectors, components each in their own part of the project) seemed like the sensible thing to do. And for simple projects, it certainly works just fine. But when I watched [a video by Dan Abramov](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers), from his [course on idiomatic Redux](https://egghead.io/courses/building-react-applications-with-idiomatic-redux), my perspective changed. I highly recommend you watch that course if you haven't already. If I don't convince you about colocating selectors with reducers in this article, I bet he can.

## A Better Way: Colocating Selectors with Reducers

Anyway, in that video Dan presents the idea of writing the selector functions in the same file where their related reducer is. I had seen this pattern before I saw the video, but I was resistant to the idea at first because it seemed to violate separation of concerns. I should have known that ever since React came along with JSX, I would be reevaluating what exactly a concern is and where it makes sense to separate it. In the case of reducers and selectors, Dan's video helped the idea finally click for me.

What is the benefit of doing it this way? By colocating your selectors with your reducer, you are encapsulating the internal structure of that slice of the state tree. The only things that need to know about how the subtree is actually structured are right there in that file, and so any future changes to that structure would only affect that one file.

Let's say your backend developers decide they need to change the shape of the user object. That never happens, right? 

