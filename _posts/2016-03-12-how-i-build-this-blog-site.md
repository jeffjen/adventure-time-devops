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
the folder where you placed this file and provision a Virtual Machine by
running `vagrant up workspace`, then `vagrant ssh workspace` to login to your
provisioned box.

Now you have a sandbox to work with, and a way to reproduce this box in case
you messed up.

Learn more on [Vagrantfile](https://www.vagrantup.com/docs/vagrantfile/)

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

{% gist jeffjen/6c2d06b99a5a0e1c41ca %}

![Provision a jekyll box]({{ site.url }}/assets/provision-jekyll.gif)

Destroy the box you created earlier by `vagrant destroy workspace`, and create
a new workspace with the new *Vagrantfile*.  The provision can take a while
depending on your machine and network performance.

With *Vagrnat*, when you feel you had a good box, you could store this box
by `vagrant package --base name-of-your-box`.  This will produce a
`package.box` object in your working directory.  Anyone can use this box by
`vagrant box add jekyll/2 /path/to/package.box`.

Learn more on [Vagrant box pacakaging](https://www.vagrantup.com/docs/virtualbox/boxes.html)
Learn more on [How to distruibute your box](information://www.vagrantup.com/docs/cli/box.html#add)

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

{% gist jeffjen/6c2d06b99a5a0e1c41ca %}

### Create a jekyll site
![Building your first jekyll site]({{ site.url }}/assets/building-your-first-jekyll-site.gif)

Login to your box by `vagrant ssh workspace`.  Make a decision on where you
want to place your site source in, here I referr to by `jekyll-sites`.

Goto `/path/to/jekyll-sites` and bootstrap a site called `your-testing-site`

- `mkdir -p your-testing-site`
- `cd your-testing-site`
- `jekyll new your-testing-site`
- `jekyll build --source your-testing-site --destination _site`

Serve the site for spot checking.  To do this, execute
`jekyll serve` under `/path/to/jekyll-sites/your-testing-site`

From your host machine browser, view the site by visiting
`http://127.0.0.1:4040`.

### Publish to Github Pages
Now that you had your source in `your-testing-site` and your generated site
`_site` under your workspace `jekyll-sites`, its time to prepare publishing.
We will publish it to [Github Pages](https://pages.github.com/) since this is
the easiest, as well as **forcing you to version control your site**.

Create a repository on [Github](https://github.com).  In your source directory
`/path/to/jekyll-sites/your-testing-site/your-testing-site`

- `git init` and perform the first commit.
- `git remote add origin [remote_url]`
- `git pull --rebase`
- `git push -u`

Now you need to prepare a dedicated branch `gh-pages`.  Github Pages takes
contents from this branch and host them on their server farm. Notice that
`gh-pages` must:

- Track files only pertain to your site.
- Keep a separate history from your source.

This is why we keep two document roots.  One for your **source code** and
**jekyll** runtime settings; the other for storing and presenting the site.

To setup, goto `/path/to/jekyll-sites/your-testing-site/_site` and setup local
repository:

- `git init`
- `git remote add origin [remote_url]`
- `git checkout --orphan gh-pages`

Add your site contents and make your first commit.  Then, push to remote
`git push -u origin gh-pages`.

And with that final step, your site is up on Github Pages.  Visit your site
through [http://your-account-name.github.io/your-testing-site/]()

So to recap the deployment steps:

- Makes changes to `your-testing-site` and commit/push your changes as you
  would handle source code.
- You can still create local branch and push to remote branch as long as the
  branch name is not `gh-pages`.
- When you are ready to publish, goto your `_site` directory and commit/push
  to `gh-pages`.
