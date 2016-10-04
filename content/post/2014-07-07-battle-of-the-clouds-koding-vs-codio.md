+++
title = "Battle of the Clouds: Koding vs. Codio"
date = "2014-07-07T21:05:00-06:00"
description = "A comparison of two cloud IDEs, Koding and Codio, and why I prefer Codio"
+++

{{% figure src="/images/post/cloud-ides/puffyclouds3.jpg" title="Image from FreeNaturePictures.com" link="http://www.freenaturepictures.com/cloud-pictures.php" alt="Photo of sunlight shining through clouds" %}}

Today I'd like to talk about two different cloud <abbr title="Integrated Development Environments">IDEs</abbr> that I've had the chance to use, [Koding](https://koding.com) and [Codio](https://codio.com), and the pros and cons of each. I know that cloud IDEs aren't really new at this point, and these two that I'm talking about certainly aren't the only ones out there, but I've only recently gotten into this game. Hopefully my perspective will help you if you're trying to decide on a cloud IDE to go with.

**Spoiler:** I prefer Codio and use it quite a bit. What you get for the price is great, and overall it seems faster and more polished than Koding.

<!--more-->

## Why Use a Cloud IDE?

As I said before, cloud IDEs aren't really new anymore---[Cloud9](https://c9.io), one of the biggest names in the industry, has been around since 2009---but only recently have I become interested in them. I had plenty of doubts at first. For one thing, I didn't like the idea of putting my data, my work, into someone else's hands. This goes along with my wariness of cloud services in general. I want to own my own data and not have to rely on some service making it available to me, not to mention having to trust that they won't, you know, hand it over to the NSA or something.

Turns out this isn't quite as big an issue as I thought, thanks to a little thing called Git. Both Koding and Codio, as well as every other cloud IDE I know of, make it straightforward to configure your projects with Git (or Mercurial, or whatever you prefer). That means you can simply add a remote repository, and now you're no longer dependent on the IDE housing your data. Codio makes it especially easy to configure repos with GitHub and Bitbucket, which I grant are yet more examples of entrusting your data to someone else, but they're pretty well respected services, and there's nothing stopping you from setting up your own private, self-hosted repository as a remote for your project.

Another major obstacle was the cost. Most cloud IDEs cost money and, like many web-based services, operate on monthly or yearly subscriptions. That's a big deal to someone who doesn't have a lot of extra money to throw around. If you're a student or just getting started in this industry, I feel your pain. When I was a student, my budget could be summed up as "Spend as little as possible." It was hard to justify paying for, much less paying *monthly* for, a service that I just thought might be cool.

So why bother with cloud IDEs at all? The biggest draw for me was portability. All you need to use these services is a web browser. That's really cool when you think about it. You get a whole environment, complete with editor and any server-side dependencies you might need, without having to install a single thing on your own computer. You don't even have to work on your own computer. You could, for example, [install a portable browser on a USB key](http://portableapps.com), plug that in at the library, and you're off to the races!

This portability is what I needed when I faced an interesting situation at work. Finding myself with some spare time, I had some ideas I wanted to experiment with that weren't part of our normal development environment. Unfortunately, the corporate firewall and proxy policies make it very cumbersome for any application other than browsers (e.g., Git, npm, Composer) to connect to the Internet. Also, since I'm used to developing on Linux for my own projects, it was difficult for me to set up the environment I needed on Windows, which I have to use at work.

A cloud IDE alleviates these problems. I don't have to install any additional software on my machine, and since everything is done through the browser, I don't have to worry about maneuvering through the proxy, thereby risking breaking company policies. (It's sad that I have to worry about that when what I'm doing will benefit the company, but such is the obstacle I face.) A cloud IDE enormously simplifies that process of setting up a project with all its dependencies so I can hit the ground running. That's what drew me in.

Now that I've talked about why you might want to use a cloud IDE, let's look at a couple of them, starting with Koding.

## Koding: Free, and Don't You Forget It!

What attracted me to [Koding](https://koding.com) is that it's free. True, it does have [paid options](https://koding.com/Pricing), but what really make Koding unique is that you can create (reasonably) unlimited public and private projects at no cost. Not bad!

When you start out with Koding, you get your own virtual machine to develop on. This is a Ubuntu server that you get full shell access to, either through their online terminal or through SSH. And when I say full shell access, I mean it---you can do absolutely anything you want on it, including stuff that requires root access.

These machines are already set up with the resources you need to do most web projects, including Apache, PHP, MySQL, Ruby, Node.js, Perl, and Python. You're certainly not limited to that list, however, since you can install anything you want on the VM. Also, out of the box you're set up with your own subdomain that follows the pattern `[username].kd.io`, and you're free to set up any number of subdomains of that subdomain that point to wherever you want on your machine.

{{< figure src="/images/post/cloud-ides/koding-hello-world.png" title="Your Koding VM is already set up with the most popular web server software." alt="Screenshot of Koding's default server page" >}}

Upon logging in to Koding, you're greeted with a tabbed interface that includes "Activity", "Teamwork", "Terminal", "Ace" (editor), and "Apps." You start out on the activity feed. Koding places a large emphasis on collaboration, especially learning how to code together. The "Teamwork" tab provides an environment for doing so, but I have to admit that I haven't used it, because this aspect of the service doesn't really interest me. From what it looks like, though, you can start up a Teamwork session, using some kind of app template if you want, and then invite others to work on it with you through the activity feed. Then you can work on the project together in real time and send chat messages back and forth. It seems like a cool idea, although something I haven't really had use for yet.

The editor is pretty nice. It appears to be the [Ace Editor](https://ace.c9.io), which, interestingly enough, is an open-source project developed and maintained by Cloud9. It does what you need it to, for the most part, but if you're coming from a more advanced desktop editor like Sublime Text, you might find yourself missing some features.

{{< figure src="/images/post/cloud-ides/koding-ace.png" title="Koding's Ace Editor" alt="Screenshot of Koding's Ace Editor" >}}

Overall, Koding is a pretty good development environment, and I'm frankly stunned that they give you access to all these resources for free.

But there are some downsides. The biggest problem I've had with Koding is server uptime and availability. It seems like every other time I log in to Koding, people on the activity feed are complaining about not being able to access their VMs. When I try to open a terminal, sure enough, it just gets stuck on the loading spinner indefinitely. I understand that Koding's user base is growing rapidly, making it a challenge to scale their resources accordingly. I also really shouldn't complain since I'm not paying them anything, but it certainly is hard to want to use Koding when it's a gamble whether or not it's going to work for me today.

Similarly, the interface seems a bit laggy a lot of the time. Now, I don't think any web-based IDE is ever going to be as responsive as working on your own machine (and anyone who tells you otherwise is probably lying), but the latency in Koding gets frustrating at times. Your VM can take quite a while to boot up, and both the terminal and the editor can show noticeable lag. I've heard that [it's the same when you access your VM through SSH](http://oldpapyrus.wordpress.com/2014/02/17/koding/). To be fair, it's been a few weeks since I've really used Koding, so things might have changed, but that was my impression.

Finally, the VMs on Koding run Ubuntu 13.04, [which reached end of life on January 27 this year](https://wiki.ubuntu.com/Releases). It may have been the latest Ubuntu version when Koding was released to the public, but I think it was a strange choice not to go with a long-term support release. I assumed that once 14.04 (which is LTS) was released that Koding would upgrade, but two-and-a-half months later, I have yet to see that happen.

If you take a look at `/etc/apt/sources.list` on your Koding VM, you'll see that the official Ubuntu repositories have been replaced with some mirrors provided by Koding themselves, so if you run an upgrade with APT, you'll get no warning about your Ubuntu version no longer being supported. I suppose it's not a huge deal, but it still bothers me. I've even tried, [along with others](https://koding.com/Activity/how-can-i-upgrade-my-vm-ubuntu-1404-i-execute-next-command-but-repositories-return), to force an upgrade to 14.04, but it never went well.

### Koding: Summary

#### Pros

*   Free for unlimited public and private projects.
*   Full shell access to your VM (including root).
*   VM comes preconfigured with software for most web projects.
*   Easy collaboration.
*   Decent editor.

#### Cons

*   Servers are unreliable.
*   Interface can be laggy.
*   Uses outdated, unsupported Ubuntu version.

## Codio: A Polished Experience at a Reasonable Cost

When I realized that Koding might not be the best for me, I started looking around at other options. Out of all the cloud IDEs I looked at (check out [this spreadsheet](https://docs.google.com/spreadsheet/ccc?key=0An6rx68cKNFNdDNYdFdSSTNzZXl5eGRSY0ZxSW10aHc&usp=drive_web#gid=1) for a partial list), Codio had the [pricing plan](https://codio.com/s/pricing) that made the most sense to me.

Codio is free for public and open-source projects. For just eight dollars a month, you get *unlimited* private projects. [Compare that to Cloud9](https://c9.io/site/pricing/), where you pay 19 dollars a month and are still limited to six private projects. In addition, Codio gives you one private project on the free plan. That was enough to get me to try it out.

When you log in to Codio, you first see a dashboard where all your projects are listed. One major difference between Koding and Codio is that in Codio, every project has a separate "box," or virtual machine, so the configuration on one won't affect the others. (If you're worried about having to re-create the same environment over and over again for different projects, Codio [recently introduced a box-cloning feature](https://codio.com/blog/clonable-boxes/) that takes the pain out of duplicating environments.) Select a project, and in just a few seconds your box will boot up and be ready to go.

To create a new project, you click on the "Create Project" button, and you're presented with a number of options for getting started. You can use a template such as [HTML5 Boilerplate](https://html5boilerplate.com), clone a Git or Mercurial repository, import a Zip file, connect to a server via (S)FTP, or use Salesforce (no idea how that works). You can choose to start a blank project under "templates."

{{< figure src="/images/post/cloud-ides/codio-create-project.png" title="You have a lot of options when creating a project in Codio." alt="Screenshot of Codio's create project page" >}}

The actual IDE feels almost like a desktop application, both in responsiveness and in features. The window is divided up into panels, which you can organize and resize however you like, and those panels contain tabs, which contain the actual stuff you work with, like files, terminals, preferences, and so on. Tabs can be moved around from panel to panel.

The Codio editor features a command bar, made popular by Sublime Text, which makes it easy to access any command or menu option through the keyboard.

{{< figure src="/images/post/cloud-ides/codio-ide.png" title="Codio's IDE is fast, full-featured, and flexible." alt="Screenshot of Codio's IDE" >}}

Each project has a configuration file where you can set up how to preview your project through the web. You can set up both static and dynamic preview URLs, and then these options are available through a convenient menu. One cool feature with the preview URLs is that if the server is listening on a non-standard port, you can access it at either `[subdomain].codio.io:[port]` or `[subdomain]-[port].codio.io`. That means if you have a proxy blocking every port besides 80 and 443, you can still access it. When you use the second URL option, Codio will route you to the right place. Pretty cool!

I could go on and on about Codio's features (in their own words, they have "More features than hairs on Chuck Norris chest [sic]!"), but let's turn now to some potential downsides. The only ones I can think of aren't really that serious, and new features are being added all the time, so the downsides might not even be around for too long.

First of all, unlike Koding, you don't get root access to your VMs on Codio, so you can't install whatever the heck you want. However, Codio mostly makes up for this with a feature called [box parts](https://codio.com/s/docs/boxes/box-parts/), which is a sort of package manager that can install [pretty much anything you'd need](https://github.com/codio/boxparts/tree/master/lib/autoparts/packages) for a web project, right into your home directory. The team also takes requests for additional box parts.

Second, I've noticed a bug with the tooltips. The editor has a nice feature where it shows tooltips with descriptions of variables and functions as you type them. The only problem is that sometimes those tooltips don't go away. They can get pretty irritating, and the only way I've found to get rid of them besides refreshing is to delete their DOM elements from the browser developer tools.

There are also some features that I'd like to see, like [EditorConfig](http://editorconfig.org) support and Sublime-style snippets. But I'm confident that those features will show up eventually. The Codio team pays close attention to [feature requests](http://forum.codio.com/category/feature-request). In fact, soon after I signed up for Codio, I got an e-mail from Ian Jobling, who I believe is the lead developer. We then had a nice chat on IRC (#codio channel on Freenode) about my experience so far with the product, and he explained to me how to get my feature requests noticed by the team. The people at Codio clearly care about what their users want.

I suppose the cost is potentially a downside. If you don't feel comfortable plopping down eight bucks a month for a service like this, I totally understand. There are free options out there. But one thing I've discovered lately is that money isn't the only cost of something. Sometimes, when something is free, you pay for it in other ways. In my case, the cost of Codio is worth it.

### Summary: Codio

#### Pros

*   Free for public projects, $8/month for unlimited private projects.
*   Isolated environments for each project, which can be cloned.
*   VMs that boot up quickly.
*   Full-featured, responsive editor.
*   Access to non-standard ports, even if your network blocks them.

#### Cons

*   $8/month for private projects.
*   No root access to VMs (but remember box parts).
*   Bug where tooltips don't go away.

## Conclusion

A year ago, I didn't think I would be saying this, but I think the whole cloud IDE idea is pretty cool, and more than that, it's quite useful. That said, I still don't use cloud IDEs exclusively. I still prefer to develop locally when I can. I still love my Sublime Text (although I'm slowly getting wooed by Atom). And it's just fun to hack on my own computer.

But sometimes I don't have my own computer with me, and that's when a cloud IDE really comes in handy. And when it comes to cloud IDEs, I think Codio is a great choice.
