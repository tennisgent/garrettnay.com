+++
date = "2017-06-30T08:00:00-06:00"
title = "Redux, Ramda, and Pointfree Selectors"
description = "Ramda helps you organize your Redux selectors in a clean and sensible way."
+++

I love Redux. Big deal, right? [A lot of people do](https://github.com/reactjs/redux/stargazers). But seriously, Redux provides a great system for managing state in complex JavaScript applications. Not only that, but it was also the first Flux-like library that I really felt I understood.

Although Redux is not directly tied to any particular view library, you'll typically find it in React applications. And much like React, the Redux library itself doesn't prescribe a specific way of organizing your app (aside from a few basic requirements). You're free to put your reducers, action creators, selectors, middleware, etc., wherever they make the most sense for you and your app (you don't even absolutely need any of those things except reducers). That's a good thing, because every app has different needs.

But that's not to say there aren't some best practices you could benefit from learning and applying.

