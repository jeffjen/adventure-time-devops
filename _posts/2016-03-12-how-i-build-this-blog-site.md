---
layout: post
title: "How I build this blog site"
date: 2016-03-12 10:00:00 +0000
categories: vagrant
tag:
 - devops
 - vagrant
excerpt_separator: <!--more-->
---

Before I started writing this blog I spent a good week surveying solutions to
my blogging workflow.  I did not opt for blog site providers such as Blogger or
Wordpress: not because they are bad, but out of weighing the tradeoff between
adapting a new workflow and tool chains I grow accustomed with.

In the end I landed on a solution using:

- Writing in Github Flavored Markdown in [Vim](http://www.vim.org/)
- Blog site generation through [Jekyll](https://jekyllrb.com/)
- Hosting provided by [Github Pages](https://pages.github.com/)
- And finally, **clear documentation** on how to customize

<!--more-->

## For the faint of heart

This post covers a great deal in how to provision an environment to start using
jekyll.  The target audience really is people interested in VM provisioning and
DevOps principles.

Check out
[Setup a Jekyll workspace and Strats for Deployment]({% post_url 2016-03-13-setup-jekll-workspace-and-deploy-strats %})
for a gentle introduction on Jekyll and Github Pages.

## Considerations
I am a DevOps engineer working with dozens of Linux machine on-premise, Cloud
Providers (AWS, Azure, etc), and on this very machine I work on.  For many
DevOps engineer, what better way to manage these systems then working in remote
login sessions?  Adept in *text based* tool chain is necessary, thus **Vim**.

Another concern I have is with [minimizing footprint](#minimizing-footprint) on
the machine I work on, and [repeatable environment](#repeatable-environment) in
case of migration or disaster recovery.

Think about the role you play in:

- As an Application Developer you work on a single project with a set of
  predictable requirements.
- As a DevOps engineer you work with many projects with different sets of
  requirements.

Finally, I need to [version control configuration](#version-control-configuration).

### Minimizing footprint
Think about what you are allowed to do when you rent a place.  You probably
will not take down a wall, retile the floor, or even place a nail on the wall.
That is because your landlord had a contract with you stating you cannot, and
rightly so for your own safety and their protection.

Treat your device like you are renting it.  Unlike renting a house, you can
create a Virtual Machine in your host machine, and make whatever changes there,
without fear of breaking your host.

How do you create a Virtual Machine?  The path of least resistance is through
[Vagrant](https://www.vagrantup.com/) +
[VitrualBox](https://www.virtualbox.org/).

Here is a simple but extendable
[Vagrantfile](https://www.vagrantup.com/docs/vagrantfile/) for staring your
first Virtual Mahcine.

{% gist jeffjen/6c2d06b99a5a0e1c41ca %}

![Provision a Vagrant + Vbox]({{ site.url }}/assets/provision-a-vagrant-vbox.gif)

Place this in a path you control, such as your `Docuement` folder.  Move to
the folder where you placed this file and provision a Virtual Machine.

{% highlight bash %}
mkdir -p /path/to/Docuement
# Create Vagrantfile from gist, or create your own
curl -sL -O https://gist.githubusercontent.com/jeffjen/6c2d06b99a5a0e1c41ca/raw/768b4695c921aa41a1a89bf3d6be7d3358db4b82/Vagrantfile
# Initialize your workspace
vagrant up workspace
# Wait until initialized, then login
vagrant ssh workspace
{% endhighlight %}

Now you have a sandbox to work with, and a way to reproduce this box in case
you messed up.

### Repeatable environment
Taking your living space as an example, consider the cost of relocating to a
different house.  Putting aside the cost of the rent, you need to package your
belongings, find a way to ship those packages, and unpack them when they
arrive.  The cost is huge, tedious, and error prone.

Now what if you need to move your virtual living space?  This consideration is
valid because:

- You had a hardware failure.
- You had an obscure failure in the Virtual Machine.
- You are upgrading your device for better performance.

The best way to recover from these problems is to create a **repeatable
environment**, here I am using *Vagrant* provisioning to reliably create an
environment with [jekyll](https://jekyllrb.com/) installed

{% gist jeffjen/b466006f3a67f91a7a81 %}

![Provision a jekyll box]({{ site.url }}/assets/provision-jekyll.gif)

Destroy the box you created earlier by `vagrant destroy workspace`, and create
a new workspace with the new *Vagrantfile*.  The provision can take a while
depending on your machine and network performance.

With *Vagrant*, when you feel you had a good box, you could store this box to
use later, or share to your organization for a consistent environment.

{% highlight bash %}
cd /path/to/Docuement
# Download the ridculously complex Vagrantfile to provision a box with jekyll
curl -sL -o Vagrantfile https://gist.githubusercontent.com/jeffjen/b466006f3a67f91a7a81/raw/8cdaba026b8029b7d967160af7b63f5230a1cf58/Vagrantfile-jekyll
vagrant up workspace
# Review VirtualBox console for the name of your VM box
vagrant pacakge --base name-of-your-box
# Add this box with desired alias
vagrant box add jekyll/3 package.box
{% endhighlight %}

### Version control configuration
I cannot stress this enough: **version control everything**.  You will be glad
when you could traceback to a working version of your environment.  Also, what
better place to store these configuration then in VCS providers such as
[Github](https://github.com/) and [Bitbucket](https://bitbucket.org/)?

## That was... kind of brutal, for setting up for Jekyll
It was.  It goes to show how much were taken for granted by people
managing your site, and perhaps they do deserve to be paid for their service.

But know this: *You pay for what you don't know*.  What you pay with today
reading this post is with your time, and if you are running a business, perhaps
you are paying with dough.

## In the next post
[Setup a Jekyll workspace and Strats for Deployment](/adventure-time-devops{% post_url 2016-03-13-setup-jekll-workspace-and-deploy-strats %})
