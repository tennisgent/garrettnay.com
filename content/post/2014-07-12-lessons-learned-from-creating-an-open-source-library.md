+++
title = "Lessons Learned from Creating an Open-Source Library"
date = "2014-07-12T09:29:19-06:00"
description = "I recently released a simple jQuery plugin, and I learned a lot of valuable lessons in the process."
+++

This past week I released [a simple jQuery plugin on GitHub](https://github.com/garrettn/jquery-takeout). It's nothing special; in fact, I don't imagine very many people ever needing it. But I learned quite a few valuable lessons during the development process---not just about writing jQuery plugins specifically, but about creating, developing, and releasing a software product and publishing its source code for all the world to see.

I'm not necessarily going to say anything groundbreaking here. You've probably heard many of them before. In fact, I already knew a lot of these things at least on an intellectual level, but there's definitely something different about learning from real experience. I hope these little lessons I've learned will be helpful to some of you.

<!--more-->

## 1. It's best to start small

I have a lot of ideas for coding projects. Some are pretty grandiose, with all sorts of features, testing suites, API integrations, and so on. But projects like those are pretty hard to get started on, especially for someone like me who is still breaking into the field.

This jQuery plugin was the perfect opportunity for me to get my feet wet. It does just one simple, stupid thing and doesn't have a lot of customization options (for now). It was therefore easy to try things like test-driven development, build tools, and continuous integration. Because the stakes were so low, I wasn't afraid to step into the unknown.

That's what I'd recommend if you're starting out in programming. You may have lofty goals for some killer apps that will change the world, but you might want to start with a smaller idea so you can get a feel for good practices and workflows without dealing with too much complexity. With those building blocks in place, you'll then be better prepared to take on bigger projects.

## 2. Automation is awesome

Automation makes software development much more enjoyable. And we have so many resources available to us nowadays.

To get started with my plugin, I used the [jQuery plugin generator](https://github.com/yeoman/generator-jquery) for the scaffolding tool [Yeoman](http://yeoman.io), which got me started with most of the structure I needed. I did have to do some tweaking, but it was much easier than starting from scratch. I then relied on [Grunt](http://gruntjs.com), the JavaScript task runner, to automatically take care of my code linting, testing, and building tasks. I even created a custom release task that [bumps the version number](https://github.com/vojtajina/grunt-bump) in all the relevant JSON files; builds the release versions of the plugin; [updates the changelog](https://github.com/btford/grunt-conventional-changelog); and commits, tags, and pushes the release to GitHub, all in one go.

With all this automation in place, I can focus on the development of the plugin without worrying about the mundane tasks. You should do the same thing. Automate as much as possible, so you can save your energy for the fun stuff.

## 3. Testing brings confidence

I've read a lot about test-driven development (TDD), but this was my first chance to really put it into practice. And I'm so glad I did.

In TDD (or <abbr title="Behavior-Driven Development">BDD</abbr> or whatever---I'm not going to distinguish), you write a test for what you want the code to do, *and then* you write the code to make it happen. Essentially, you can't add a feature without writing a test for it first.

The benefits to this approach are many. One is that it forces you to think about what the code *really* needs to do, so you can leave out the extra, unnecessary stuff. It makes you develop in smaller, (hopefully) better-structured increments. But my favorite benefit is the confidence that comes from knowing for sure whether any change you make will affect your code's ability to function properly. If you've written your tests well (not that I claim to be great at this yet), and you make some change that breaks your code, you will know right away because your tests tell you.

Even if you don't like writing the tests first, making sure everything has tests really pays off.

## 4. It's not going to be perfect

This point goes along a bit with the first one. I think we all know intuitively that no product is going to be perfect, but you truly have to accept that fact to make any progress, especially when working in open source.

I had been hesitant for a long time to put any code on GitHub, because it's scary to put stuff out there for everyone to see. There are some brilliant programmers out there, and if they saw my work, surely they would ridicule it or just wave it off as the work of an amateur. And yes, my code is amateurish compared with a lot of theirs, but how can I expect to get any better if I don't try to collaborate?

Isn't that the point of open source---to freely share our ideas with each other so that they can get even better and everyone can benefit from them? Maybe one day someone will actually find my plugin useful, and they will come up with some improvements that will make it awesome. That can't happen if I don't make the plugin available.

If you wait for your product to be perfect before you publish it, you'll never produce anything.

## 5. Sometimes it really is your fault

This lesson wasn't so fun to learn. I don't know about you, but when there is a problem with something I've coded, my initial reaction is "The person testing must not know what they're doing" or "There must be something wrong with the system." And often I turn out to be completely wrong.

For example, at one point my tests started timing out. It was really strange because they only timed out when I ran the tests through Grunt, but when I opened the test page in the browser, they worked just fine. I thought there must be something wrong with PhantomJS on my machine, so I decided to just push my changes anyway and see if the tests worked on [Travis CI](https://travis-ci.org/garrettn/jquery-takeout). They failed there as well.

It didn't make any sense. I looked at the diff between the commit where the tests passed and this latest commit, and the only changes I made were to the documentation. Something was obviously wrong with the system. Except there wasn't.

In reality, the problem started in the *previous* commit, where I changed the package name in `package.json`. That package name is used everywhere in the Gruntfile, including the URLs that PhantomJS opens to run the tests. Because had changed the package name, PhantomJS was trying to open URLs that didn't exist, hence the timeouts.

In short, it was my fault. If I had run the tests before making that previous commit (I didn't think it would matter because I wasn't changing any "real" code), I would have caught the problem. But I didn't. I screwed up.

Sometimes that happens. Bugs show up in the code, and they are there because of something stupid you did. That's okay. It happens. You fix it and move on. But you can't fix it if you're not open to the idea that you might have done something wrong. Remember, it's never going to be perfect.

## 6. There is inherent value in creating something

I've said a couple times that nobody else may find my little plugin useful. I created it for a very specific situation at work, and I don't know if anyone else will ever face a similar situation. Maybe nobody will ever even *look* at my plugin.

But that doesn't mean it was a wasted effort. I'm glad I did it, because it's very fulfilling to create something. I think if you ask some of the best novelists, painters, or even filmmakers, they'll tell you they're not creating stuff to get rich or famous. They're creating because they have an irresistible urge to create; fame or fortune is just a nice side effect.

The same principle applies to any area where you create something, including programming. I know we have to make a living, and I'm sure it's gratifying to see something you've worked on become enormously successful. But if you don't find inherent joy or satisfaction in the process of making something, even just a tiny bit once in a while, I think you're missing out.

That's one thing I love about open source. People are releasing amazing tools and resources without any expectation of monetary reward. They do it because they love doing it and because they hope they can make the world just a little better.

It's a great thing to be a part of, even if it's just by creating one small, simple jQuery plugin.
