+++
title = "Still Not Sure about AngularJS"
date = "2014-08-19T08:22:47-06:00"
description = "My thoughts and concerns about the AngularJS framework"
+++

I had originally planned to title this post "I'm Starting to Get Sold on AngularJS," but a blog post I read the other day got me thinking again. In ["Opinionated Rundown of JS Frameworks"](https://blog.andyet.com/2014/08/13/opinionated-rundown-of-js-frameworks), [Henrik Joreteg](https://twitter.com/HenrikJoreteg) from [&yet](http://andyet.com) gives some interesting critiques of Angular (as well as other JavaScript frameworks) that caused some of my old doubts to resurface. I still think Angular is a very cool framework, and I intend to continue experimenting with it, but I'm hesitant to treat it like the only serious player in the game, as so many developers seem to do already.

<!--more-->

In his post, Henrik gives his opinion on the pros and cons of AngularJS, and you can tell he feels pretty strongly about it because the cons are much longer and more detailed. I'd like to offer my comments on some of those cons, including my original doubts and how those doubts have evolved.

## 1. Learning Angular vs. Learning JavaScript

Henrik's first concern with Angular is that developers run the risk of gaining skills that are very Angular-specific and not easily translatable to other contexts:

> Picking Angular means you’re learning Angular the framework instead of how to solve problems in JavaScript. ... I’ve got developers who’s [sic] primary skill is Angular, not necessarily JavaScript.

My first reaction to this point was to ask, How is that any different with any other framework? I believe that any time you use a framework to build an application, you're going to have to learn and use some skills that apply to that framwork only. You can't abstract away forever.

But the truth is that Angular is a pretty different beast next to all the other JavaScript frameworks out there. It uses a module system all its own. It chucks a lot of Web standards in favor of creating its own HTML "directives" to define DOM behavior. It uses dependency injection for *everything*.

None of these things is necessarily bad (especially the dependency injection). But I can see the risk of becoming so ingrained with "the Angular way" that you don't know how to do things outside of an Angular app.

I myself resisted learning Angular for a long time. I got wowed by demonstrations at conferences like a lot of people, but I didn't want to put serious effort into learning how to develop with the framework, I think because 1) it's trendy, and I have a natural resistance to trendy things; 2) it's backed by Google, which I have a bit of a . . . distaste for (more on that in a future post, possibly); and 3) I didn't want to change, Backbone being my first love in the world of JS frameworks.

Obviously, none of these reasons are good excuses for not expanding my skills and trying new things. I also realized that a lot of companies use AngularJS for their apps, so if I want to stay relevant, I at least have to learn how to use it. So lately I've been learning what I can.

Turns out, I think Angular is pretty dang cool. It's amazing what you can create with very little setup and configuration. I'm also really interested in the [Ionic Framework for mobile apps](http://ionicframework.com), which integrates tightly with Angular.

Ironically, as I started learning Angular, I felt the opposite of what Henrik is expressing here. I believed that working with Angular instead of Backbone would make me a better overall JavaScript developer, mainly because Angular itself doesn't provide any sort of model abstraction. Instead, you usually work with plain old JavaScript objects (POJOs), which requires you to really understand how JavaScript's prototype paradigm really works. But now I'm starting to see what he means---when you consider the framework as a whole, you can really get used to a certain way of developing that relies mainly on Angular skills, as opposed to JavaScript skills.

So is learning Angular worth it? Most definitely. But I want to be careful not to let that be the only way I know how to develop wep applications.

## 2. Separation of Concerns

I have gone back and forth on this issue a lot. Henrik says that Angular violates the principle of [separation of concerns](http://en.wikipedia.org/wiki/Separation_of_concerns) because of its HTML directives:

> In Angular you spend a lot of time describing behavior in HTML instead of JS. For me personally, this is the deal breaker with Angular. I don’t want to describe application logic in HTML, it’s simply not expressive enough because it’s a markup language for structuring documents, not describing application logic.

I totally felt the same way at first. When I saw those `ng-click` directives, I thought, How are those any different from the `onclick` HTML attributes that are so looked down upon nowadays?

There's at least one big difference, actually, and that is that the expressions inside Angular directives are always contained within the scope of their controller, whereas the `onclick` attribute operates within the global scope. The whole concept of scope in Angular is pretty cool.

```html
<!-- This click handler calls the global addItem function -->
<button onclick="addItem(item)">Add</button>

<!-- This one calls the addItem function that exists on the $scope object of the controller -->
<button ng-click="addItem(item)">Add</button>
```

But there's still the issue of application logic inside the HTML. That does go against what you're traditionally taught about web development. It's funny, though, because Angular developers consider this a major selling point of the framework. And a short while ago I realized, Why not? Why not let the HTML worry about the HTML, so the JavaScript can worry about the data and business logic? On the other hand, it seems bad to let *some* of the application logic go into the HTML while most of it stays in the JavaScript.

The jury is still out for me on this one. I think both sides make valid points, and I guess what really matters is which approach helps you accomplish the task more effectively.

## 3. Magic

No doubt about it, AngularJS has a lot of [magic](http://en.wikipedia.org/wiki/Magic_%28programming%29). That's what makes it so great for demos. Now some people in the programming community will tell you that magic in a framework is a great thing, while others will say it's a terrible thing. Henrik seems to lean toward the latter camp:

> Magic comes at a cost. When you’re working with something that’s highly abstracted, it becomes a lot more difficult to figure out what’s wrong when something goes awry. ... I would guess most Angular users lack enough understanding of the framework itself to really feel confident modifying or debugging Angular itself.

That describes me pretty well. I am still fairly new to Angular, though, and I have to say that the magic of Angular is pretty impressive. The two-way data binding is, of course, awesome. But one of the coolest features for me is how the dependency injection works. The injector knows what to give you just by what you call the parameter to your function! I have never seen that anywhere else in the JavaScript world.

```javascript
angular.module('MyApp')
    .controller('MainCtrl', function ($scope, $http) {
        // It knows what $scope and $http are just because you named them that!
    });
```

But as Henrik says, magic comes at a cost. When there is so much happening for you automatically, you have to wonder how much functionality you're including that you don't actually need, which could be a drain on performance. Angular is increasingly separating out its functionality into modules, though, so that may become less of an issue.

Even more to the point, the more magic your framwork provides, the easier it is to remain unaware of what goes on under the hood, making it harder to tailor to your specific needs and harder to optimize. This point goes along with the first one, in that all this magic increases your reliance on the specific framework you use.

That isn't to say that you should avoid any magic or abstraction that frameworks provide (in fact, Henrik makes this point later on in his post). Frameworks exist to make it easier for us to develop better applications more efficiently. They also make it a lot more fun (and Angular is definitely fun to work with). But too much abstraction can be a danger because, in Henrik's words, "when you veer off the beaten path, you’re on your own."

## Conclusion: Have I Gotten Anywhere?

I think I've proven the premise of the title of this post---I'm just not sure how I feel about AngularJS. It has been fun to learn and explore, certainly. And I think it's important to understand how to develop applications with it on some level at least. Angular obviously isn't going away any time soon. More and more development teams are turning to it, and if I want to be a part of some of the cool work that is going on, I'm going to have to speak the language.

Still, I'm afraid to go all in and become an Angular developer through and through. Instead I'll focus on becoming a great, well-rounded JavaScript developer who knows how to select and use the best tools for the job at hand.

Perhaps not surprisingly, at the end of the post I've been referring to, Henrik discusses a new framework created by the people at &yet called [Ampersand](http://ampersandjs.com). Of course they're going to favor their own creation, but I've been looking at Ampersand the past couple days, and I have to say, it is pretty cool. It's like the new generation of Backbone. I will certainly have more to say about it in a future post.

Until then, keep on learning and keep on coding!
