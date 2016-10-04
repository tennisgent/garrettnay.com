+++
title = "Amazing Resource DevDocs Goes Open Source"
date = "2013-10-25T07:39:00-06:00"
description = "Highlights of the features of DevDocs, an all-in-one documentation tool, and why you should be using it."
+++

[DevDocs](http://devdocs.io), the all-in-one API documentation tool for web
developers, released [its code on GitHub](https://github.com/Thibaut/devdocs) yesterday, officially becoming an
open-source project. This is good news indeed. Now that anybody can contribute
documentation, this already-amazing resource is only going to get better.

If you are a web developer and aren't using DevDocs, you are seriously missing
out. Here are a few reasons why.

<!--more-->

## Lots of Documentation All in One Place

{{< figure src="/images/post/devdocs/devdocs.png" title="The DevDocs interface" alt="Screenshot of the DevDocs interface" >}}

DevDocs is a one-stop shop for (nearly) all of your documentation needs. It
includes docs for core web technologies as well as various popular libraries and
frameworks. This is the current lineup:

* **Core web:** CSS, DOM, DOM Events, HTML, HTTP, JavaScript

* **Libraries and frameworks:** Angular.js, Backbone.js, Ember.js, jQuery,
jQuery Mobile, jQuery UI, Lo-Dash, Underscore.js

* **Preprocessors:** Coffeescript, Less, Sass

* **Server:** Node.js, PHP

Do you use any of those in your development work? I thought so. Have you ever
needed to look up a particular function signature, or the correct syntax of a
CSS rule? Quit hunting around on Google and individual websites and use DevDocs!
You'll find it all here.

For some reason, DevDocs has been a bit of a hard sell to my co-workers. When I
eagerly showed one of them how all of the documentation we need is here in a
single place, he just shrugged it off and said when he needs to look at jQuery
documentation, he goes to the jQuery site, and he knows all the JavaScript
anyway. That second point may be true (it's definitely not for me yet), but
what if he needs to check what parameters that PHP function takes, or whether
that jQuery function is deprecated? Why go to the separate sites when you can
get it all in one place?

One thing I probably should have stressed when showing it to him is that
**DevDocs contains exact copies of the official documentations.** Although they
may be styled a little differently, the content is exactly the same, and so you
don't need to worry about missing something from the official site. It's all
there.


And if you're worried that all those documentations will clutter up your search
results with things you don't need, worry no more. DevDocs allows you to select
only the docs that you want to use and ignore all the others. You can tailor it
to your specific work!

{{< figure src="/images/post/devdocs/select-docs.png" title="Select only the documentation you want" alt="Screenshot of the list of documentations you can download in DevDocs" >}}

## Lightning-Fast Fuzzy Searching

This feature is really what sells it for me. From what I've seen, DevDocs beats
any official website's search speed hands down. Results come up instantly as you
type, and then you can quickly navigate to the right one with the keyboard and
open it up in the main pane.

But what's really great is that it supports **fuzzy matching**. If you use
[Sublime Text](http://www.sublimetext.com/), you know how great fuzzy matching is. What it means is that
you can type in a string of letters that aren't necessarily contiguous, and it
will understand what you mean. Say, for example, I want to look up CSS
transforms. I can just type in `tfm`, and "transform" will show up as the first
entry. Select that one, and I'm there. The whole thing took two seconds.

{{< figure src="/images/post/devdocs/search.png" title="Namespacing your searches is easy" alt="Screenshot of the DevDocs search functionality in action" >}}

It's also possible to namespace your searches. As I said before, DevDocs has
  lots of different sets of documentation. If you want to, say, look up information
about jQuery events, you can type in `jq` and then a tab. A "jQuery" label will
appear in the search field, and from then on your searches will be restricted to
the jQuery documentation (you can use backspace to get rid of the label and
restore the scope).

Oh, and you can also query straight from the URL, with our without namespacing.
Simply add `/#q=` and your query to the end of the app's URL, and you'll be
taken to the app with the search field prefilled, with the first result showing
up in the main pane. To get a namespaced result, simply prefix your query with
the namespace followed by a space. For example, `devdocs.io/#q=js trim` takes
you to [the JavaScript trim function documentation](http://devdocs.io/#q=js%20trim).

However you like to search, I'm telling you DevDocs is fast.

## Now It's Open Source

DevDocs has been out in the wild for several months now, but only yesterday did
Thibaut Courouble, the developer, release the code for all to see and contribute
to. This was a big step, and one that he clearly put a lot of thought into.
There are several benefits to DevDocs being an open-source project.

For one thing, now it is possible to download your own copy of DevDocs and run
it locally, meaning you don't need an Internet connection for it to work. I
haven't tried doing that myself, but it could be a great option for some
developers.

The more obvious benefit is now anybody can contribute to the project, both by
improving the code of the app itself and by adding new documentation. Thibaut
has given [guidelines for contributing](https://github.com/Thibaut/devdocs/blob/master/CONTRIBUTING.md), so the way is now open for anyone
to help. I expect that soon we'll be seeing new documentations being added at an
accelerated rate, and that can only be a good thing.

## Use DevDocs Already!

I could talk about other features of DevDocs, such as its mobile interface,
keyboard shortcuts, and OpenSearch compatibility, but hopefully what I've
outlined here gives you reason enough to start using it. Seriously, I can't
think of any reason why a web developer wouldn't want to use it--unless you had
every bit of every documentation memorized.

So [get over there](http://devdocs.io) and start using DevDocs to help your work.
I guarantee it will make you happier as a developer.
