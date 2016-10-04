+++
title = "Octopress: Deploy to GitHub, Back Up to Bitbucket"
date = "2013-10-03"
description = "Deploy your Octopress blog publicly to GitHub Pages and back it up privately to Bitbucket."
+++

It's probably pretty obvious that this blog is built with [Octopress](http://octopress.org).
Obvious at least until I use a new theme.

Octopress is built on top of the static site generator [Jekyll](http://jekyllrb.com), and together
they make a fantastic blogging platform for hacker types. You can write your
posts in your favorite editor in plain text, you can track your changes with
Git, and you can publish on any web server that can serve static HTMl, CSS, and
JavaScript---no database necessary!

One popular place to deploy an Octopress blog is on [GitHub Pages](http://pages.github.com), and
Octopress has built-in functionality to make it trivially easy to do.<!--more-->
The blog you're reading right now is a testament to how simple it is (as is the
fact that the "Octo" in Octopress refers to the GitHub mascot, Octocat, I
believe.)

## Setting Up Deployment to GitHub

The first thing you need to do, obviously, is create an account on GitHub. When
you have an account set up, you then create a repository that takes the form
`username.github.io`.[^1] GitHub will automatically recognize this repo as your
personal site, and any static web content you push here will be published.

Next, if you haven't already done so, [clone the Octopress repository on your computer and install the dependencies](http://octopress.org/docs/setup/).

Once Bundler finishes installing all the gems, you'll have the `rake` command
available to you with lots of tasks already configured to help you customize,
preview, generate, and deploy your blog.

The Rake task I want to focus on here is the aptly named `setup_github_pages`.
Simply invoke that task to get started.

```
$ rake setup_github_pages
```

It will now ask you for the URL for your repository. You can use either the
Git or HTTPS protocol, both of which should be easy to find on your
repository page. For this example, I'll use the URL for my own GitHub pages
repo.


```
Enter the read/write url for your repository
(For example, 'git@github.com:your_username/your_username.github.io.git)
           or 'https://github.com/your_username/your_username.github.io')
Repository url: git@github.com:garrettn/garrettn.github.io.git
```

And that's all you need to do! You should now see a bunch of output like this:

```
Added remote git@github.com:garrettn/garrettn.github.io.git as origin
Set origin as default remote
Master branch renamed to 'source' for committing your blog source files
rm -rf _deploy
mkdir _deploy
cd _deploy
Initialized empty Git repository in ......./garrettn.github.io/_deploy/.git/
[master (root-commit) 2d8c9ba] Octopress init
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
cd -

---
## Now you can deploy to git@github.com:garrettn/garrettn.github.io.git with `rake deploy` ##
```

A couple important things have happened here:

1. The task added your repo URL as a Git remote and named it "origin." If you
had cloned Octopress from the official repo, that remote was renamed to
"octopress," and it's still available so you can conveniently pull in changes
from upstream.

2. The master branch, which was up to this point called "master," was renamed to
"source."

3. A new `_deploy` directory was created, and inside that directory a new Git
repository was created with that same remote as "origin." This folder will be
ignored by Git in the root of your project, but here is where the "master" branch
is that will actually be pushed to your repository on GitHub.

Now, like the script says, you simply need to enter `rake deploy` to make the
magic happen. You can run this command right after you generate your site.

```
$ rake generate
$ rake deploy
```

When you invoke the `generate` command, your site will be generated into the
`public` directory. The `deploy` task will then go into the `_deploy` directory
and pull in any changes there may be on the server (there shouldn't be). Next, it
will copy the contents of the `public` directory into `_deploy` and commit the
changes to the master branch. Finally, it will push that branch up to your remote
repository on GitHub.

Within about 10 minutes, you should see shiny new blog at `username.github.io`.
That was easy, wasn't it?

## What About the Source Files?

As I said before, the Rake task for setting up deployment changes your main
branch from "master" to "source." Since the deployment task only pushes the
master branch, it's completely up to you what to do with the source. You'll want
to back it up somewhere, and probably the easiest thing to do is to push that
branch up to GitHub also.

```
$ git push -u origin source
```

But suppose you don't want your source files to be publicly accessible? It might
be a silly thing to worry about, since everything in your source files gets
published to the site anyway. But what if you are working on a post that isn't
quite ready for prime time? Jekyll lets you set an option in the YAML front
matter of a post called `published`. When it is set to `false`, you'll still
see the post when you're previewing your site locally, but it won't get
generated for deployment.

So if you're working on a post that's going through several revisions, you might
not want to make it publicly accessible yet, but you'll probably still want to
back it up somewhere. What to do?

One of the simplest solutions is Bitbucket.

## Backing Up to Bitbucket

[Bitbucket](https://bitbucket.org) is another code hosting service, perhaps not as popular as GitHub,
but still awesome and here's why: **unlimited free private repositories**.
Whereas GitHub charges you for any amount of private repos, Bitbucket bases its
payment scheme on how many users you have collaborating on your repos (it's
free for up to five users). So if you have a private project that you want to
back up to a safe place but not where it can be accessed by anybody, Bitbucket
is an excellent resource.

So here's what you do: Create a Bitbucket account if you don't have one, and
then set up a new repository for your Octopress source files (I gave it the same
name as on GitHub). On the page for your new repository, you'll find a URL that
you can add to your local Git repository. Then you simply add that remote. I
called mine "backup."

```
$ git remote add backup ssh://git@bitbucket.org/garrettnay/garrettn.github.io.git
```

Now, whenever you have commits you want to back up, simply push the source
branch up to Bitbucket.

```
$ git push -u backup source
```

(Use the `-u` option the first time to set up a tracking branch. After that it's
no longer necessary.)

And that's all there is to it! Now you have an easy way to deploy your blog,
and also a place to back it up away from prying eyes. It's a setup that's worked
really well for me.

See, I told you Octopress was fun.


[^1]: I'm not going to go into details here about how to create repositories on GitHub or Bitbucket. The process on both sites is pretty straightforward, and they both have excellent documentation.
