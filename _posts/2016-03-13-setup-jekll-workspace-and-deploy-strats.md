---
layout: post
title: "Setup a Jekyll workspace and Strats for Deployment"
date: 2016-03-13 10:00:00 +0000
categories: jekyll
tags:
 - jekyll
 - github
 - vagrant
excerpt_separator: <!--more-->
---

The best place to learn how to use jekyll is starting with their
[excellent guide](https://jekyllrb.com/docs/usage/).  The goal for this
post is to teach you how to review and publish a jekyll site.
Nonetheless, I will show you the minimum steps required to start one.

This post covers the following topic:

- Blog site generation through [Jekyll](https://jekyllrb.com/)
- Hosting provided by [Github Pages](https://pages.github.com/)

<!--more-->

## Before you start
If you had missed the first post about
[How I build this blog site]({{site.baseurl}}{% post_url 2016-03-12-how-i-build-this-blog-site %})
you might want to check that out first.  But if you are interated only in
learning how to manage Jekyll and Deployment Strategy,
[skip ahead](#create-a-jekyll-site)

Here is the template Vagrantfile you will be using, notice that I had already
added the box we provisioned earlier as `jekyll/3`:

{% gist jeffjen/e170fa06e6ef892cb77c %}

## Create a jekyll site
Make a decision on where you want to place your site source in, here I refer
it by `jekyll-sites`.

{% highlight bash %}
# Bootstrap a new site called your-testing-site
mkdir -p /path/to/jekyll-sites/your-testing-site
cd /path/to/jekyll-sites/your-testing-site
jekyll new your-testing-site
# Generate site from source
jekyll build --source your-testing-site --destination site
# Serve the site for spot checking.
jekyll serve -s your-testing-site -d site -H 0.0.0.0
{% endhighlight %}

View the site by visiting `http://127.0.0.1:4000` from your host machine.

![Building your first jekyll site]({{ site.url }}/assets/building-your-first-jekyll-site.gif)

## Publish to Github Pages
Now that you had your source in `your-testing-site` and your generated site
`site`, its time to prepare publishing.

We will publish to [Github Pages](https://pages.github.com/) since this will
**force you to version control your site**.

Create a repository on [Github](https://github.com), then from your
`jekyll-sites`:

{% highlight bash %}
cd /path/to/jekyll-sites/your-testing-site/your-testing-site
# Initialize your site source and configuration
git init
# Add remote repository URL and pull + rebase
git remote add origin remote_url
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
**jekyll runtime settings**; the other for **presenting the site**.

{% highlight bash %}
cd /path/to/jekyll-sites/your-testing-site/site
# Initialize generated contents and assets
git init
# Add remote repository URL
git remote add origin remote_url
# Checkout an orphaned branch gh-pages
git checkout --orphan gh-pages
# Add your site contents and make your first commit, then push to remote.
git push -u origin gh-pages
{% endhighlight %}

And with that final step, your site is up on Github Pages.

Visit your site through [http://your-account-name.github.io/your-testing-site/]()

So to recap the deployment steps:

- Direct Content and configuration changes to `your-testing-site`.
- Manage your branch like you normaly would, provided not named `gh-pages`.
- When you are ready to publish, goto your `site` directory and commit/push
  to `gh-pages`.
