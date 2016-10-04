+++
title = "Keep All Your Octopress Plugins in One Place"
date = "2013-10-07T15:57:00-06:00"
description = "Deploy your Octopress blog publicly to GitHub Pages and back it up privately to Bitbucket."
+++

Since I'm still getting this blog started and learning the ropes with Octopress,
in my first few posts I'm probably goint to talk about it a lot. So far it's
been quite a journey.

I'm starting to incorporate some third-party plugins [of which there many](https://github.com/imathis/octopress/wiki/3rd-party-plugins).
These plugins add some very useful functionality, and I highly recommend you
check them out.

One thing I've noticed that for several plugins (if not all--I haven't checked),
in order to install them you have to download multiple files and copy them into
different places in your project.<!--more--> There's nothing wrong with that;
it's just the way it works. But what if you want to keep track of where those
plugin files came from? What if you want to keep them up to date easily without
downloading them individually from GitHub every time there's an update? I've
found a way to address these issues painlessly (or at least less painfully)
using Git submodules.

## Installing Plugins as Git Submodules

[Submodules are a pretty funky feature of Git](http://git-scm.com/book/en/Git-Tools-Submodules) that allow you to include
another repository as a sub-project of your main repository, keeping track of
their relationship to each other while still keeping them independent. Weird,
right?

Let me illustrate. Let's say I want to replace Octopress's default Google site
search with the awesome service from [Tapir](http://tapirgo.com). Wouldn't you know it,
[there's a plugin for that](https://github.com/blimey85/octopress-tapir) (and a mighty fine one if I do say so myself!).
But instead of just downloading the files and copying them as instructed in the
readme, I'm going to add this repository as a submodule of my blog repository.

Here is how I add it as a submodule:

```
$ git submodule add https://github.com/blimey85/octopress-tapir .plugin-sources/tapir
```

It's essentially the same syntax as `git clone`. I'm cloning it into a directory
called `.plugin-sources/tapir`, which will be created if it doesn't already
exist. I'm calling it `.plugin-sources` to differentiate it from the `plugins`
directory, which contains only Ruby scripts for various plugins. The dot is there
to keep it hidden and consistent with the `.themes` directory.

The output of that command looks pretty much the same as what you'd see with
`git clone`. But when I enter `git status`, the result is different.

```
$ git status
# On branch source
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   .gitmodules
#	new file:   .plugin-sources/tapir
#
```
There's a new file called `.gitmodules`; this is where Git lists information
about each submodule that you've installed. Also notice that the
`.plugin-sources/tapir` directory is there, but it's listed as a file. As far
as my master repository is concerned, that directory is nothing but a single
file, and it doesn't care what goes on inside that repository.

Until there's new commit in it, that is. You see, the submodule is a special
type of file that points to a specific commit of the repository. If the
repository changes what commit it's on, either from your own work or from pulling
in updates, the outer repository of my project will notice and consider it a
modified file. More on that in a bit.

First, I will commit my changes and push them to [my backup repository]({{< relref "post/2013-10-03-octopress-deploy-to-github.md">}}).

```
$ git commit -m "Add Tapir plugin"
$ git push backup source
```

There! Now that plugin is saved in my repository, and I can copy the necessary
files out of it. And if I ever need to work on my blog on another machine, I
just do a recursive clone to include all the submodules with it.

```
$ git clone --recursive ssh://git@bitbucket.org/garrettnay/garrettn.github.io.git
```

## Updating the Plugins

So let's say at some point in the future, I'm strolling around GitHub and I find
out that the Tapir plugin has been updated. I want to incorporate those updates
into my blog, and since I've installed the plugin as a submodule, it's easy to
do.

All I have to do is go into the plugin directory and use Git to pull in the
changes from GitHub.

```
$ cd .plugin-sources/tapir

$ git pull
## OR ##
$ git fetch origin
$ git merge origin/master
```

Now I have the latest version, and I can copy the updated files into my project.

When I go back into the root directory of my blog, Git tells me that the
submodule has been changed. Specifically, there are new commits on that
repository.

```
$ git status
# On branch source
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   .plugin-sources/tapir (new commits)
#
no changes added to commit (use "git add" and/or "git commit -a")
```

This basically means that submodule is pointing to one commit, but the repository
itself is now pointing to a different (newer) one. To get the submodule lined
up with the repository, I just to a `submodule update` command.

```
$ git submodule update
Submodule path '.plugin-sources/tapir': checked out 'e20ce325efa29ca9deb0ac952085b3cf27462275'
```

The hash will be different, depending on the commit. But the submodule has now
been updated, and I am all set to use the new plugin!

So that's how I manage my plugins with Octopress. One issue I haven't addressed
is the fact that I still have to manually copy new or updated files into my
project directory. I might try using symbolic links instead, but I would need to
experiment with that first. This system isn't perfect, but at least it is one
step above manually downloading individual files from the project repository.

How do *you* like to manage your Octopress plugins?
