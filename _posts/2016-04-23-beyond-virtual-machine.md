---
layout: post
title: "Beyond Virtual Machine, into Containers"
date: 2016-04-23 10:00:00 +0000
categories: docker
tag:
 - devops
 - docker
excerpt_separator: <!--more-->
---

Consumers today have two options (three if you bother the hassel) when
purchasing device for work: either you have one with **OS X** installed, or
**Windows** installed (and for the hard core fans, your choice of **unix
distro**).  This creates a problem for application developers because the
environment you work and test in is different from your deployment target,
usually a unix/linux box.

For those working with windows server, don't even bother with your complaints.
[Look at where Microsoft is heading](https://www.linux.com/news/bash-windows-what-does-it-mean).

For any developers worth their salt, you would have already learned how to work
with Virtual Machines.  With the advent of [Vagrant](https://www.vagrantup.com/),
provisioning on localhost had been made *hassel free*.  If you have never heard
of such wonderful creation, go look at
[How I build this blog site]({{site.baseurl}}{% post_url 2016-03-12-how-i-build-this-blog-site %})
for a tour.

## The Problem

Still with using Virtual Machine the biggest pain point is the amount of
resource spent when dealing with many boxes at once.  This problem occurs often
when you work as a DevOps where mapping your test environment to match
staging/production environment is necessary.  You could purchase VMWare
solutions or build your own OpenStack infrastructure, but really is this
necessary?

## The Answer

The answer is [Docker](https://www.docker.com/).

<!--more-->

Instead of explaining to you the technology of
[Docker](https://www.docker.com/), I opt to demonstrate use cases where Docker
eliminates the need for Virtual Machine, and make your work life better.

But first, lets setup an environment to run Docker.  Two choices here:

- Install using [Docker Toolbox](https://www.docker.com/products/docker-toolbox)
- [Build a Virtual Machine with Docker installed.](#for-the-brave-tinker)

## What can I do with Docker

### Running program in an isolated environment
Node.js, Ruby, and Python apps depend on an interpreter to operate.  If you are
running an application written in those language only for the feature it
provides, there is very little reason to install the interpreter on the system.

For example, I am not a Ruby developer, but I want to use
[jekyll](https://jekyllrb.com/) for blog generation; instead of installing
Ruby which I will rarely use, package jekyll as a Docker Image.

{% gist fc8739efcd89c4557005cde416731506 %}

- Copy the contents to `Dockerfile` in your workspace.
- Run `docker build -t jekyll .`

#### What had just happened?

- Docker Daemon pulled down `ruby:latest` image from [Docker Hub](https://hub.docker.com/)
  to your disk
- Docker Daemon builds a new image called `jekyll:latest`  
    - From the specifications in `Dockerfile`
    - `ruby:latest` as base image.

List the images by running `docker images`.

#### How do I run jekyll?
{% highlight bash %}
# Download the helper script
curl -sL -O https://gist.githubusercontent.com/jeffjen/fc8739efcd89c4557005cde416731506/raw/74712ff2c508e56d43840650c6c9f6efd3a05312/jekyll
# Make it executable
chmod +x jekyll
# Launch jekyll to serve and watch files
./jekyll serve -s your-blog-src -d your-blog-site -H 0.0.0.0
{% endhighlight %}

Visit your site from your host machine at `127.0.0.1:8080`

#### Why is this better?

- You did not install Ruby for just one app.
- The Linux Distro you are using might not have Ruby 2.x available.  To get
  around this you need [rvm](https://rvm.io/) to fetch and build Ruby from
  source.

### Developing in the target environment
I am a [Golang](https://golang.org/) enthusiast; Golang by itself deserves a
dedicated topic and **should be picked up by any DevOps who wants to ship
compact artifacts**.  But in order to build and write in Golang I need the Go
developer tool chain installed on my host machine to compile/build Go code...
right?

**No**.  You could write the code in your natrual environment.  When you are
ready to build, build with Docker.

{% highlight bash %}
docker run --rm -v ${GOPATH}:/go -w /go/src/path-to-project golang:1.6 \
    go build -v
{% endhighlight %}

#### What is this mad magic

- You command Docker Daemon to  
    - Create and start a one time `container` using `golang:1.6` image.
    - If the image is not there already it is downloaded.
- You bind directory `${GOPATH}` on your host to `/go` in the `container`
- You set the runtime working directory to be where your project is.  
    - Original path is `${GOPATH}/src/path-to-project`
    - We used `/go` as anchor when specifiying directory in the `container`.
- Finally, the command you run in the `container` is `go build -v`.
- Your binary appears in `${GOPATH}/src/path-to-project`

#### No like v1.6!!
Fine.  Change the image version to `golang:1.5`.

Consider the implication: Suppose you are working with `Python 2.7`, but there
are projects built with special requirements from `Python 3`.  Installing and
managing different Python versions is a dangerous task that could break your
system if not handled properly.

Avoid these problems with running or building with Docker.

### A shippable environment
You might have picked up the trend here with the introduction of Ruby and
Golang base images.  These images are built and maintained by the
Docker Hub team so that users could repliably reproduce an environment with
Ruby or Golang installed.

In the same vein, you could build and ship full applications as Docker images.
Take the `jekyll` case for example, you do not need to rebuild the image on
each machine; build the image and push to Docker Hub, then pull the image to
other machines that requires it.

But a Docker image can be not just an application package.  You could ship *the
whole environment* as an image, bells and whistles included, such that people
in your organization works with consistent tool chain.

Take my own [Dockerfile](https://github.com/jeffjen/workspace/blob/master/Dockerfile)
as an example:

- Common tools installed
- Bind mounts and configuration installed
- Ready to use environment

Now when I move to a new host machine, I can restore my workspace by

- Bootstrap the machine to run Docker
- Pull my workspace image and launch  
    - Golang, Node.js, Python, and Ruby installed
    - Command line utility tools installed
    - Text editor installed and configured
    - Shell environment installed and configured

## For the brave tinker
For those of you who want more control over your Docker Environment, here is a
step by step guide using **Vagrant**.

{% gist jeffjen/a3d4830c06b716ae26c4 %}

{% highlight bash %}
# Move to your desired workspace
mkdir -p /path/to/Docuement
# Download Vagrantfile
curl -sL -o Vagrantfile http://bit.ly/Vagrantfile-docker
# Provision the box
vagrant up docker-workspace
{% endhighlight %}

### How to configure Docker Daemon
Docker Daemon by default listens on Unix Domain Socket `/var/run/docker.sock`.
This prevents accidental provisioning from unwanted party.

Since we are *professional* DevOps and we know what we are doing, let us
configure the Daemon to listen on TCP and Unix Domain Socket.

{% gist jeffjen/cd96d92d0318f58e9901138d08851f85 %}

{% highlight bash %}
mkdir -p /etc/docker
curl -sL -o /etc/docker/daemon.json http://bit.ly/docker-daemon-cfg
{% endhighlight %}

### Assume you are working with debian/ubuntu
{% highlight bash %}
curl -sL -o /etc/default/docker http://bit.ly/docker-default-options
{% endhighlight %}

### Assume you are working with CentOS/RHEL
{% highlight bash %}
curl -sL -o /etc/sysconfig/docker http://bit.ly/docker-default-options
{% endhighlight %}

Once the configuration files are in place, restart the Docker Daemon:
{% highlight bash %}
service docker restart
{% endhighlight %}

### Download the command line Docker client
Now that the Daemon is available on port `2375`, we can start using docker
client from your **host machine**.

{% highlight bash %}
# Download and install docker client
curl -sL -O https://get.docker.com/builds/`uname -s`/`uname -m`/docker-1.11.0.tgz
tar xvf docker-1.11.0.tgz
cp docker/docker /usr/local/bin/docker
# Test connection to Docker Daemon
docker -H localhost:2375 version
{% endhighlight %}
