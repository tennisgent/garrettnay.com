+++
title = "Putting the “Minimum” in “Minimum Viable Product”"
date = "2016-11-11T00:30:00-07:00"
description = "Introducing garrettnay.com, my new personal website."
+++

Hello there. Welcome to garrettnay.com.

If you're viewing this close to the time I published it, you may be experiencing dreadful flashbacks to the early days of the World Wide Web, back before CSS, back before even table layouts were a thing. But the CSS didn't fail to load. The CSS isn't broken.

There is no CSS. <i>Cue dramatic music</i>.

Why am I doing this? First off, let me be clear that I *do* intend to make this site look nice eventually. Default browsers styles certainly aren't much to look at. But since I was making this transition to a new site with a new sytem powering it anyway, I thought I'd try something a little different: build the site up gradually, all out in the open.

<!--more-->

## Goodbye Octopress, Hello Hugo

You can't have a new blog without talking about the engine running it, right?

For a few years I published my old site on GitHub Pages using Octopress 2, which was essentially a layer of customization on top of the popular static site generator Jekyll intended to make tech blogging easy. Indeed it did make it easy to get started, and I was quite happy with it for a good while. But [as I mentioned in a post a couple years ago]({{< ref "post/2014-06-10-the-obligatory-redesign-post.md" >}}), eventually I wanted to get away from it. At the time I was mostly concerned about customizing the theme. But other problems cropped up as well. Setting up on a fresh system was kind of a pain. I ran into issues trying to deploy sometimes. Images in my posts would randomly disappear. Stuff like that.

To be fair, Brandon Mathis himself [has said that he's well aware of the problems with Octopress 2](http://octopress.org/2015/01/15/octopress-3.0-is-coming/), and he's been creating Octopress 3 in a completely different way. But I got tired of waiting and started looking elsewhere. I compared a lot of different static site generators, especially the ones based on Node. What I eventually settled on wasn't Node-based, though. Say hello to [Hugo](http://gohugo.io).

It turns out Hugo is also pretty popular. I was hesitant at first, not knowing any Go, which Hugo is built with, but so far that has not been a limitation. One nice thing about Hugo is that it's installed a standalone executable—no Ruby or Node package managers necessary! It's also insanely fast. Ruby and Node definitely can't compete on speed with this thing.

But what I really like about it is how flexible it is. It doesn't assume anything about your content at the beginning and gives you virtually zero boilerplate. It does expect you to organize your content in a certain way, but it's still pretty configurable. As far as the theme goes, it gives you nothing to start with. You can install third-party themes, but you can also start completely from scratch.

## Building from Scratch, Out in the Open

In the web development world there seems to be an ongoing debate of whether it's better to start a new project with some kind of starter kit or boilerplate or to build it up on your own, piece by piece. The answer to that question depends a lot on the nature of the project. If you're on a tight deadline, using a boilerplate might be the way to go. But if you really want to learn something thoroughly, nothing substitutes building from scratch.

I see this site as an opportunity for me to learn. Not just to learn Hugo, but to learn how to build a website in general. I've been working as a web developer for years now, but I still think there is much for me to learn or re-learn. How to write truly semantic and accessible HTML. How to write CSS that performs well and follows good design guidelines. How to use responsive images. How to take advantage of emerging technologies like HTTP/2 and Service Workers. This site will be my playground, my research lab.

And hey, it's my project, so I can do this however I want.

Therefore, I'm starting from square one, more or less. I'm publishing this site with HTML that is minimal and hopefully semantic and accessible. I'll tweak it as I learn more. I'll start adding CSS and judiciously applied JavaScript. This site will grow as I grow. And there's no reason to hide it. The whole world can see my progress.

## Open-Sourcing All the Things

To really drive the point home, I'm also making the source code of this website available [on GitHub](https://github.com/garrettn/garettnay.com). In [one of my first posts]({{< ref "post/2013-10-03-octopress-deploy-to-github.md" >}}) I explained that I preferred to keep the source of my site (i.e. the input to the static site generator) in a private repository in case I was working on a post I wasn't ready to share yet. I've changed my mind. I never actually ran into the situation I was talking about, and even if I do, I don't see any harm in having a post up on a branch in the repository before it's actually published. If anyone does come across it (which is also doubtful), maybe I could get some valuable feedback from them.

Also, maybe someone can look at the source of my site to find ideas on how to build their own Hugo site. I don't pretend to be a Hugo expert by any means, but as I learn, hopefully I can help others learn too. That's one reason we have open source, right? To help each other.

## This Will Take Time

Please be patient with me as I embark on this new project. I'm working on it in my spare time, which has become a very limited and precious commodity in my life lately. There's obviously a lot of work to be done here, including a lot of functionality that I had on my old site. I'm also still figuring out a strategy for pointing all my old blog URLs over to this new domain. But I'm excited about the challenge, and I hope to learn and share some valuable lessons along the way.

Thanks for visiting!
