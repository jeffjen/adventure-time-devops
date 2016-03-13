---
layout: post
title: "How I build this blog site"
date: 2016-03-12 10:00:00 +0000
categories: docker jekyll
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
use later, or share to your organization for consistent environment.

{% highlight bash %}
cd /path/to/Docuement
# Review VirtualBox console for the name of your VM box
vagrant pacakge --base name-of-your-box
# Add this box with desired alias
vagrant box add jekyll/2 package.box
{% endhighlight %}

### Version control configuration
I cannot stress this enough: **version control everything**.  You will be glad
when you could traceback to a working version of your environment.  Also, what
better place to store these configuration then in VCS providers such as
[Github](https://github.com/) and [Bitbucket](https://bitbucket.org/)?


## About that jekyll site building...
The best place to learn how to use jekyll is starting with their
[excellent guide](https://jekyllrb.com/docs/usage/).  The goal for this
last section is to teach you how I review and publish my jekyll site.
Nonetheless, I will show you the minimum steps required to start a one with
the box you created.

Here is the template Vagrantfile you will be using, notice that I had already
added the box we provisioned earlier as `jekyll/2`:

{% gist jeffjen/b466006f3a67f91a7a81 %}

### Create a jekyll site
![Building your first jekyll site]({{ site.url }}/assets/building-your-first-jekyll-site.gif)

Login to your box by `vagrant ssh workspace`.  Make a decision on where you
want to place your site source in, here I referr to by `jekyll-sites`.

Goto `/path/to/jekyll-sites` and bootstrap a site called `your-testing-site`

{% highlight bash %}
# Bootstrap a new site called your-testing-site
mkdir -p /path/to/jekyll-sites/your-testing-site
cd your-testing-site
jekyll new your-testing-site
# Generate site from source
jekyll build --source your-testing-site --destination site
{% endhighlight %}

Serve the site for spot checking.

{% highlight bash %}
cd /path/to/jekyll-sites/your-testing-site
jekyll serve -s your-testing-site -d site -H 0.0.0.0
{% endhighlight %}

View the site by visiting `http://127.0.0.1:4000` from your host machine.

### Publish to Github Pages
Now that you had your source in `your-testing-site` and your generated site
`site` under your workspace `jekyll-sites`, its time to prepare publishing.
We will publish it to [Github Pages](https://pages.github.com/) since this is
the easiest, as well as **forcing you to version control your site**.

Create a repository on [Github](https://github.com).

{% highlight bash %}
cd /path/to/jekyll-sites/your-testing-site/your-testing-site
# Initialize your site source and configuration
git init
# Add remote repository URL and pull + rebase
git remote add origin [remote\_url]
git pull --rebase
# Optional push if you have had commited changes
git push -u
{% endhighlight %}

Now you need to prepare a dedicated branch `gh-pages`.  Github Pages takes
contents from this branch and host them on their server farm. Notice that
`gh-pages` must:

- Track files only pertain to your site.
- Keep a separate history from your source.

This is why we keep two document roots.  One for your **source code** and
**jekyll** runtime settings; the other for storing and presenting the site.

{% highlight bash %}
cd /path/to/jekyll-sites/your-testing-site/site
# Initialize generated contents and assets
git init
# Add remote repository URL
git remote add origin remote_url
# Checkout an orphaned branch gh-pages
git checkout --orphan gh-pages
{% endhighlight %}

Add your site contents and make your first commit, then push to remote.

{% highlight bash %}
git push -u origin gh-pages
{% endhighlight %}

And with that final step, your site is up on Github Pages.

Visit your site through [http://your-account-name.github.io/your-testing-site/]()

So to recap the deployment steps:

- Direct Content and configuration changes to `your-testing-site`.
- Manage your branch like you normaly would, provided not named `gh-pages`.
- When you are ready to publish, goto your `site` directory and commit/push
  to `gh-pages`.
