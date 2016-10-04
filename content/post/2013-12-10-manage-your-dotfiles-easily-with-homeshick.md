+++
title = "Manage Your Dotfiles Easily with Git and Homeshick"
date = "2013-12-10T15:31:03-07:00"
description = "How to use Homeshick/Homesick to keep track of dotfiles and synchronize them between computers."
+++

It's been a while since my last post, and one reason for that is that I spent
some time upgrading to Ubuntu 13.10 and configuring my new system. It may be a
bad idea, but for some reason I prefer doing a fresh install rather than directly
upgrading when a new version of Ubuntu is released. But as fun as it can be to
configure a shiny new system, I've learned that I really need a more systematic
way to keep track of some of the more repetitive tasks. One very useful tool I
found for doing this is called [Homeshick](https://github.com/andsens/homeshick).

<!--more-->

That's not a typo: Homeshick is a tool similar to the better-known [Homesick](https://github.com/technicalpickles/homesick),
but instead of being a Ruby gem, it's written in pure Bash---hence the "sh." It
took me a while to decide which one to use, but since I need my dotfiles set up
before I can start using Ruby (since I use [rbenv](https://github.com/sstephenson/rbenv)), it makes more sense for
me to use the version that takes nothing more than running a shell script.

Homeshick makes it very easy to track your important dotfiles and synchronize
them between computers.

## What Are Dotfiles, and Why Should You Track Them?

Dotfiles are, simply put, files that begin with a dot. On UNIX-like operating
systems, like GNU/Linux and OS X, dotfiles are hidden by default. That means
your file manager and your command line shell won't show them unless you
specifically tell them to.

For example, in Ubuntu's file manager you can hit <kbd>ctrl+h</kbd>, and you'll
see files like these:

{{< figure src="/images/post/homeshick/hidden-files-nautilus.png" title="Hidden files in Nautilus" alt="Screenshot of Nautilus file manager showing hidden files" >}}

On the command line, you can use the <kbd>-a</kbd> flag with the <kbd>ls</kbd>
command to show hidden files in the current working directory:

```
user@computer:~$ ls -a
.              .compiz    Dropbox          .ICEauthority  .pki           .steampid       .Xauthority
..             .config    .gconf           .icons         .profile       Templates       .xsession-errors
.adobe         .dbus      .gem             .local         Public         .thunderbird    .xsession-errors.old
.bash_aliases  Desktop    .gitconfig       .macromedia    .pulse-cookie  tmp
.bash_history  Dev        .gksu.lock       .mozilla       .pyenv         Ubuntu One
.bash_logout   .dmrc      .gnome2          Music          .rbenv         .unison
.bashrc        Documents  .gnome2_private  .npm           .ssh           .vagrant.d
Books          Downloads  .grabMyBooks     .nvm           .steam         Videos
.cache         .dropbox   .homesick        Pictures       .steampath     VirtualBox VMs

```

That's great, but what do these dotfiles *do*? Well, all sorts of things. I
showed [in a previous post]({{< relref "post/2013-10-21-run-programs-from-your-home-directory-in-ubuntu.md" >}}) how some files, like `.profile` and `.bashrc`, let
you set environment variables. But that's just the beginning. You can set Bash
aliases, define functions, customize the prompt, and more. And those are just
the files dealing with the shell. Some dotfiles deal with specific programs,
like Git or npm.

The bottom line is that if you're a developer, you're very likely to find
yourself customizing many of these files to make your work easier (or *possible*,
even). And if you happen to work on multiple machines (such as at home and at
work), or if you migrate to a new machine or OS like I did, you're going to find
it very tedious to keep editing these files over and over to make them the same.

There has to be a better way, right? You're in luck! There is a better way---it's
called Homeshick.

## Installing Homeshick

Since Homeshick is just a Bash script (or a small collection of scripts), you
don't really need to install it like a regular application. You just need to put
it in a place where you can access it. All it takes is a simple Git clone:

```
$ git clone git://github.com/andsens/homeshick.git $HOME/.homesick/repos/homeshick
```

This command simply clones the Homeshick repository to a hidden directory inside
your home directory. An added benefit of using Git to clone it is that you can
keep it up to date easily with a <kbd>git pull</kbd>.

Notice how the top-level directory is called `.homesick`. The developer
[says this is intentional](https://github.com/andsens/homeshick/wiki/Tutorials#bootstrapping) for compatibility with Homesick.

Now you have access to the Homeshick script. You can start using immediately by
sourcing it:

```
$ source "$HOME/.homesick/repos/homeshick/homeshick.sh"
```

This command will load a `homeshick()` function into your shell session. Of course,
you'll want to have access to this function every time you open a shell session
without having to source it manually, so you should add the line to your `.bashrc`:

```
$ printf '\nsource "$HOME/.homesick/repos/homeshick/homeshick.sh"' >> $HOME/.bashrc
```

Now you'll be able to call `homeshick` every time you open up the terminal.

## Tracking Your Dotfiles

Homeshick and Homesick both revolve around what they call "castles"—a reference
to the saying "A man's home is his castle." A castle is a Git repository where
you store dotfiles. It has a particular structure, but you don't need to worry
about that because you can have Homeshick create one for you.

```
$ homeshick generate dotfiles
```

Here, I created a new castle called dotfiles, but you can call them whatever you
want. You could even have separate castles for different purposes, like one for
Bash files, one for Git configurations, and so on. Go crazy!

For me, so far, having one central castle is enough. So once you have a castle
created, you can start telling Homeshick to track whatever dotfiles you want.
How? You guessed it—with <kbd>homeshick track</kbd>.

```
$ homeshick track dotfiles .bashrc
```

Here you're telling Homeshick to track your `.bashrc` (since you've customized
it to include Homeshick and want to save the changes) in your `dotfiles` castle.
As a result, Homeshick moves the file into the castle's directory and then
creates a symbolic link in its original location pointing to it.

What's so great about that, you ask? As I said before, the castle is a Git
repository, so now that the actual file is inside it, you can take advantage of
Git to keep track of your changes to it—and easily back it up to a remote
location. For example if you [search for dotfiles on GitHub](https://github.com/search?q=dotfiles&ref=cmdform) you'll come up
with roughly seven bazillion results. Developers love their dotfiles repos! I,
myself, don't feel the need to publish my own dotfiles for the world to see, so
I host mine in a private repository on Bitbucket. Whatever you do is up to you.

To look inside your castle, you can use a shortcut provided by Homeshick:

```
$ homeshick cd dotfiles
```

Unless you've altered the code somehow, this is the same as typing
<kbd>cd ~/.homesick/repos/dotfiles</kbd>. But the Homeshick version is nicer,
wouldn't you say?

And now that you're inside the directory, you can do all the normal Git stuff.
In this example, since you just started tracking `.bashrc` you'll see it here
as a new file staged for committing:

```
$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   home/.bashrc

```

Notice how the file is inside `home` directory. That's how Homeshick and Homesick
both structure their castles.

## Never Feel Homes(h)ick Again

So that's the basics of getting started with Homeshick. Once you've pushed your
castle to a remote repository, it's easy to incorporate those dotfiles into a
new system. Homeshick even provides a <kdb>clone</kdb> command for that purpose.

I've barely scratched the surface of what you can do with Homeshick—see
[the official documentation](https://github.com/andsens/homeshick/wiki) for more details. The point is that Homeshick
makes it easy to keep track of those precious dotfiles. And they really are
precious to a developer, as I'm beginning to discover!

Don't forget, also, that since this tool uses Git, you can harness Git's
capabilities to make your dotfiles repo fit your needs. Specifically, remember
branching. My main computer runs Ubuntu, but a while back I was thinking about
trying out a different Linux distribution on another machine. At first I thought
I would have to create a separate castle for that distro, but then I
remembered—Git has branching! You can have a branch for Ubuntu, a branch for
Arch, a branch for OS X, or whatever, and thus take advantage of the similarities
between systems while preserving the differences, *and* keep it all in one
place. It's beautiful.

So there you go. Homeshick. Use it. Or maybe you have your own preferred system
for tracking your dotfiles ([and there are many](http://dotfiles.github.io/)). What have you found works
for you? Let me know in the comments.

[8]: http://dotfiles.github.io/
