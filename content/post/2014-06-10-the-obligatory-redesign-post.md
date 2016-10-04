+++
title = "The Obligatory Redesign Post"
date = "2014-06-10T07:00:48-06:00"
description = "I've delayed writing my blog because I was looking at upgrading or switching to another platform, but I'll just stick with Octopress 2.0 for now."
+++

Hi. Yes, I'm still here. And check out my new design! It may not look like much, but it represents a big step for me.

You see, ever since I started this Octopress blog, I always had this nagging feeling that I should do something about the design. The default Octopress theme is nice, but nobody wants to stick with the default theme, right? It's like how Twitter Bootstrap looks nice, but it's so nice that for a while it seemed as if half of the Web were using it. You could spot a Bootstrap-based site immediately (still can, I suppose). And you could pin my blog as an Octopress blog immediately, too. Not that that was a bad thing, but I didn't want this to get written off as "yet another Octopress blog."

I wanted to create a new theme. Oh, I had big plans. I envisioned adapting one of the awesome templates by [HTML5 Up](https://html5up.net), and eventually open-sourcing it so everyone could easily integrate it into their own Octopress blogs. But then some things started happening in [the Octopress world](https://twitter.com/octopress) that put me in stop-and-wait mode for quite a while.

<!--more-->

## Octopress 3?

A few months ago, I learned that [Brandon Mathis](https://twitter.com/imathis), the creator of Octopress, is working on version 3.0 of the project. This isn't just an evolution of version 2---it's a completely new way of working. It's so different, in fact, that he created [a new repository](https://github.com/octopress/octopress) for it. Octopress 3.0 is a Ruby gem rather than just a Git repository that you clone and modify, which is how 2.0 works. And all its various components are separated out into individual gems, including the way [themes and plugins](https://github.com/octopress/ink) work.

And when I heard about this new version, it sounded like it was going to have a stable release very soon, so I thought, cool, I would wait and see how the whole thing worked, and then I could migrate my blog and apply my new theme using the new system.

Fast forward a few months, and Octopress 3.0 still isn't *quite* done. To be more precise, the Octopress gem is pretty much there, but Octopress Ink, the plugin and theme framework, is still being worked on. I believe it's almost ready, but the documentation is still very sparse, and I don't think I have the time or the energy to dig into it and figure out how it works for myself, especially since I know very little Ruby.

Don't get me wrong. I don't blame Brandon at all for having this take longer than planned. That's just how things go sometimes, and he clearly wants this to be the best product it can be. But because of the long delay, I started thinking about turning to other solutions.

## What About This Assemble Thing?

Perhaps you've heard of [Assemble](http://assemble.io), the static site generator for Node.js. It's essentially the Node counterpart of Ruby's Jekyll, which Octopress is based on. It works as a Grunt plugin, and it even has a Yeoman generator, the bottom line being that it uses technologies that I'm much more familiar with.

I was excited about Assemble for a while. Since it uses Grunt, a tool I now have a fair amount of experience with, I could potentially customize my blog to be exactly how I wanted it, including the theme. The possibilities seemed endless.

Unfortunately, the Assemble project is still fairly young, and its documentation is pretty limited. I searched long and hard for information on how I could make Assemble work as a replacement for Octopress, but all I could find was partial answers. There is also some functionality that I need that isn't quite there yet.

I know, Assemble is an open-source project---including its documentation---and instead of complaining about it I should just contribute to making it better. And I really might someday. But when it comes down to it, I have to decide: How much time and energy am I willing to invest in hacking on my blog, when simpler solutions are available?

## Sticking with Version 2 For Now

Sometimes, even with all the desire for hacking and customizing, you just want it to work. And the fact of the matter is that Octopress 2 does work for me. There are a few hiccups now and then, mainly dealing with syntax highlighting, which I hope will be improved in the new version. But overall it has been a solid blogging platform, one that is simple to set up and simple to work with.

So in the end, I've decided to stay with Octopress version 2 for the time being. I'll keep watching the progress on version 3, because I'm still eager to upgrade when the time is right. I'll also keep tabs on Assemble because I still want to take advantage of it some day, if not for this blog, then for other projects. But as for me and my dev blog, we will use Octopress 2.

Having decided that, I looked around for a theme that would work with Octopress 2. It didn't need to be anything fancy, just something different enough from the default. Turns out there are [quite a few themes](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes) available, and some have even been created quite recently. And now that I've found a theme that I'm happy with (after a few tweaks), I feel like I can keep writing on this blog without that redesign monkey on my back.

## For the Record---Theme Details

I feel I should give credit where credit is due. The design of this blog is mostly thanks to the work of people more skilled than I.

 - The theme is 95% based on [Bold and Blue](https://github.com/johnkeith/boldandblue) by [John Keith](http://www.john-keith.com).
 - The sans-serif font (modified from the original theme) is [Fira Sans](http://mozilla.github.io/Fira), an open-source font provided by Mozilla.
 - The speckled noise background in the header comes from the [SVG Patterns Library](https://philiprogers.com/svgpatterns/).
 - The social icons were provided by [IcoMoon](https://icomoon.io).

Okay, now that the obligatory redesign post is out of the way, let's move on to brighter age of dev blogging!
