+++
title = "Run programs from your home directory in Ubuntu"
date = "2013-10-21T21:18:00-06:00"
+++

<ins>**Update 23 Nov 2013:** Since I upgraded to Ubuntu 13.10, setting variables
in .pam_environment no longer works for me. No matter what I have in there, even
if the syntax is correct, I get blocked from logging in to my account. I haven't
yet figured out why this is happening or what the solution is, but until I do,
I'm setting my PATH variable in .profile instead. I recommend you do the same.</ins>

## TL;DR

Setting up a directory to run local applications is very simple. You just need
to do two things:

1. Create a directory for them. I like to use `.local/bin` in my home directory.

2. Edit the PATH in one of the files that sets environment variables. In Ubuntu,
I recommended setting it in <del>.pam_environment</del> <ins>.profile</ins>.

## Introduction

I really enjoy using Ubuntu, and GNU/Linux systems in general. In my opinion,
they're ideal for web development.

A situation I've run across recently is when I want to run applications that
exist in my user directory, rather than in the system directories. <!--more-->
These might be programs I created or compiled myself, or they might have been
installed by a package manager other than the system package manager (in
Ubuntu's case, APT).

For example, several software environments I work with have their own dedicated
package managers. Node.js has npm, Ruby has gem, and TeX Live has tlmgr, to name
a few. And while it's true that many of the packages available through these
managers can be installed from Ubuntu's main repositories, I've decided that it's
better to use the dedicated managers when they exist. That way I can be sure to
have the most up-to-date packages, and they're generally easier to work with.

I've also decided that when I'm using dedicated package managers, I want them to
install to my home directory rather than to the default system directories. This
may just be a personal preference, but I like not having to type `sudo` before
every install command. It also feels a bit safer: since I'm only making changes
to my home directory, nothing that goes wrong can be *that* catastrophic, and
the changes are easier to undo.

In this post I'm just going to focus on how I like to configure my system to run
applications out of my home directory, without going into specifics about
the different package managers. It all basically boils down to two steps:

1. Create a place to keep local applications.

2. Tell the system to look in that place when searching for applications.

## Create a Directory for Local Applications

There really is no one right place to put an application directory. It basically
comes down to what works best for you. What matters is that you have write
permissions to the directory, meaning you don't have to use `sudo` to make any
changes to it or inside it. Normally, that means inside your home directory.

I've seen several different approaches. Some people like to create a `bin` folder
right inside their home directory ("bin" stands for "binaries"). Others create
a `local/bin` structure, mirroring the `/usr/local/bin` system directory found
in the standard UNIX filesystem. Wikipedia says the following about the
purpose of the `/usr/local` directory:

{{% blockquote source="Wikipedia" link="http://en.wikipedia.org/wiki/Unix_directory_structure#Conventional_directory_layout" title="Unix directory structure" %}}
Resembles /usr in structure, but its subdirectories are used for additions not
part of the operating system distribution, such as custom programs or files from
a BSD Ports collection. Usually has subdirectories such as /usr/local/lib or
/usr/local/bin.
{{% /blockquote %}}

Based on this description, `local/bin` seems like a fitting place. This is my
choice, with one small difference: I use `.local` instead of `local`. The dot
in front of the name makes it a hidden directory, so it won't show up with an
`ls` command unless you use the `-a` flag, and it won't be visible in a file
manager unless you set it to show hidden files (usually with Ctrl+H). I like it
that way, because normally when I'm working in my home directory, I'm only
concerned about files per se and not about programs. It just makes things feel
less cluttered to me. Also, Ubuntu already happens to have a `.local` directory
in your home directory by default. Not very compelling reasons, I know, but
that's what I do.

So if you're still following along, create a `bin` directory wherever you like
and then change into it.

```
$ mkdir ~/.local/bin && cd $_
```

Now let's create a simple shell script to see if it's working. Create a file
called "tron" using the editor of your choice. Nano is probably the simplest.

```
$ nano tron
```

And, of course, you can call the script whatever you want, but I like knowing
that a program is fighting for me.

```
#!/bin/sh

echo "I fight for the users!"
```

Once you've saved the file, set it to be executable.

```
$ chmod +x tron
```

Now try calling the program. If you call it without any path, you should see an
error.

```
$ tron
tron: command not found
```

This error occurs because the shell doesn't know where to find the command `tron`.
If you tell it the exact directory to find it in, it will work. Right now, since
we're in the same directory, it's as simple as typing `./tron`. But that could
become rather annoying rather quickly if you need to remember the directory and
type it in, no matter where you are, any time you need to use the program.

Fortunately, there is a better way. What we need to do is change an environment
variable called PATH.

## Change PATH to Recognize the Directory

PATH is an environment variable that the system uses to find executables. It's
essentially a list of directories separated by colons. When you type in a
command, the shell will search through the directories listed in PATH, going
from left to right. As soon as it finds an executable that matches, it will run
it. If it doesn't find it, you'll see an error like the one above.

To see what your current PATH is, you can type `echo $PATH`. You should see
something like this, though probably not exactly like this:

```
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

Unless you've already made changes to PATH, you should see only system directories
there. What we need to do is add our custom directory to the list.

To change PATH on the fly, we can use the `export` command:

```
$ export PATH=~/.local/bin:$PATH
```

**Important note:** Whenever you modify PATH, it's important that you include a
reference to the existing PATH. Otherwise you'll overwrite the whole thing, and
you'll lose access to critical commands. References to existing environment
variables are prefixed by the dollar sign.

This command works easily enough, but as soon as you close the terminal or log
out, your changes will be forgotten. To make your changes persist between
sessions, you need to edit a file in your home directory.

Here's where it gets a little tricky. There are several hidden files in your
home directory that have the potential to change environment variables (see
also [EnvironmentVariables](https://help.ubuntu.com/community/EnvironmentVariables#Session-wide_environment_variables) from the Ubuntu wiki):

* **.bashrc**---This is possibly the simplest place to set them. It's basically a
script that gets run every time you start a bash session as a **non-login shell**
(e.g., when you open up a new terminal window). You can just add the `export`
command described above to the end of this file, and the next time you start a
bash session, it will work. But if you need to access your programs outside of
a bash shell, you're out of luck.

* **.profile**---This script is run every time you start a **login shell**. That
includes graphical environments, so when you log in to your normal desktop, this
script will run. The advantage here is that it runs only once when you log in,
rather than every time you open a terminal window (although if you're only
setting environment variables, the performance gains will be negligible). This
script is also sourced when you log in via SSH.

* **.bash_profile**---This one is similar to .profile, except it runs only in a
login bash shell (such as through SSH). Ubuntu doesn't have this file by default,
but other systems might.

* **.pam_environment**---Ubuntu is the only distro I'm aware of that uses this
file. This one is unique in that it is only for setting environment variables
(you can't run arbitrary script commands in it). It also has a different syntax
from the others. This file is read when you log in--before .profile is read, as
far as I can tell.

The Ubuntu wiki recommends setting environment variables in .pam_environment, <del>so
that's what I do as far as it is possible</del>. <ins>(**Update 11 Nov 2013:** After
upgrading to Ubuntu 13.10, I've found that having anything in .pam_environment
prevents me from being able to log in, even if the syntax is correct. Until I find
a solution, I'm now setting my path in .profile.)</ins> As I mentioned, the syntax
for this file is different. You can refer to the `pam_env.conf` man page for details
(either [online](http://linux.die.net/man/5/pam_env.conf) or by entering `man pam_env.conf`), but here are the main
things you need to watch out for.

*	Each line is a variable declaration, taking this form:

	`VARIABLE [DEFAULT=[value]] [OVERRIDE=[value]]`

	You can use either `DEFAULT` or `OVERRIDE`, or both. I just use `DEFAULT`.

*	When referencing other variables, use the form `${VARIABLE}`. Note the curly
braces--other files don't use them, but this one does.

With those caveats out of the way, you're ready to create your .pam_environment
file (there most likely won't already be one). Change back to your home
directory and create the file.

```
$ cd
$ nano .pam_environment
```

And just add the following line to the file:

```
PATH DEFAULT=${HOME}/.local/bin:${PATH}
```

What we've done here is set the default PATH to include the `.local/bin` folder
inside your home directory (the HOME variable points to your home directory),
and then the rest of the existing path. As I mentioned before, PATH is a
colon-separated list of directories. While you could technically list the items
in any order, keep in mind that the list is read from left to right, so something
in one directory will override another thing of the same name down the list.

**Important note:** Be absolutely certain that you have the syntax for this
file correct. If you don't, you may not be able to log back in after you've
logged out. The Something Groovy blog [has a horror story about this happening](http://somethinggroovy.blogspot.com/2012/07/pamenvironment-syntax-in-ubuntu.html).
So just be careful.

Once you've ensured that the syntax is correct, save the file and then log out.
Any change to this file will take effect only after you've logged out and back
in again.

After you've logged back in, commands in that `.local/bin` folder should now be
accessible to you. Go ahead and test it out by seeing if Tron is still fighting
for you. Open up a terminal and type:

```
$ tron
```

If all went well, you should see the response:

```
I fight for the users!
```

Hooray for Tron! Now you have a convenient location to put your local
applications that you can access on the command line from anywhere in the
filesystem. Enjoy!
