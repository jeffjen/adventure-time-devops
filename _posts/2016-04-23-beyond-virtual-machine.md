---
layout: post
title: "Beyond Virtual Machine, into Docker"
date: 2016-04-23 10:00:00 +0000
categories: docker
tag:
  - devops
  - docker
description: >
  With three use cases: Running Ruby app in a continer, building native program
  with a container, and a portable workspcae that could be deployed on any
  linux machine, I demonstrate why Docker will radically change how you manage
  your system.
excerpt_separator: <!--more-->
---

Consumers today have two options (three if you bother the hassel) when
purchasing device for work: either you have one with **OS X** installed, or
**Windows** installed (and for hard core fans, your choice of **unix**).  This
creates a problem for application developers because the environment you work
and test in is different from your deployment target, usually a unix/linux box.

For those working with windows server, don't even bother with your complaints.
[Look at where Microsoft is heading](https://www.linux.com/news/bash-windows-what-does-it-mean).

For any developers worth their salt, you would have already learned how to work
with Virtual Machines.  With the advent of [Vagrant](https://www.vagrantup.com/),
provisioning on localhost had been made *hassel free*.  If you have never heard
of such wonderful creation, go look at
[How I build this blog site]({{site.baseurl}}{% post_url 2016-03-12-how-i-build-this-blog-site %})
for a tour.

## The Problem
Still with Virtual Machine the biggest pain point is the amount of resource
and time spent when dealing with many boxes at once.  This problem occurs often
when working as a DevOps where mapping test environment to match
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
- [Build a Virtual Machine with Docker installed.](#for-the-brave-tinkerer)

## Nomenclature:
- A **Container** is a linux kernel feature that provides software level
  isolation.
- Container can be created, started, stoped, and removed.
- An **Image**  is the execution context.  A Container is created with an
  Image.
- **Dockerfile** is the recipe for building a Image.
- Each **RUN**, **ADD**, **COPY**, etc adds a **Layer** to the Image.
- **ENTRYPOINT** is the command that runs when the Container is started.
- **CMD** has two meaning:  
  - When an ENTRYPOINT is specified, CMD will be tacked on as arguments to
    ENTRYPOINT.
  - If we specified CMD without ENTRYPOINT, that is the command at container
    start.
  - CMD can be overrided at the command line.
- You **store and ship** the image.

## Running program in an isolated environment
Node.js, Ruby, and Python apps depend on a interpreter to work.  If you are
running an application written in those language ONLY for the feature it
provides, not to develop with it, there is very little reason to install the
interpreter on the system.

For example, I am not a Ruby developer, but I want to use
[jekyll](https://jekyllrb.com/) for blog generation; instead of installing
Ruby which I will rarely use, I packaged jekyll as a Docker Image.

{% gist fc8739efcd89c4557005cde416731506 %}

{% highlight bash %}
# Download Dockerfile for jekyll
curl -sL -o Dockerfile http://bit.ly/Dockerfile-jekyll
# Build docker image
docker build -t jekyll .
# Download the helper script
curl -sL -O http://bit.ly/jekyll-cli
# Make it executable
chmod +x jekyll-cli
# Create a new blog src if you don't have one already
./jekyll-cli new your-blog-src
# Launch jekyll to serve and watch files
./jekyll-cli serve -s your-blog-src -d your-blog-site -H 0.0.0.0
{% endhighlight %}

{% include youtube.html id="S1beABvOPCA" %}

Visit your site from your host machine at `localhost:8080`

List the images by running `docker images`.

### What had just happened?

- Docker Daemon pulled image `ruby:latest` from [Docker Hub](https://hub.docker.com/)
  to your disk
- `ruby:latest` is the base image that provides `ruby` interpreter and
  `build-essentials` for building native gems.
- We specified `jekyll` to be the `entrypoint` of the image, and the default
  command line flag is `--help`

### Why is this better?

- You did not install Ruby for just one app.
- The Linux Distro you are using might not have Ruby 2.x available.  To get
  around this you need [rvm](https://rvm.io/) to fetch and build Ruby from
  source.
- Suppose you wanted to try an alpha version of jekyll, you need only change the
  gem version and build a sepearte Docker Image.  **No rollback worries**.

## Developing in the target environment
I am a [Golang](https://golang.org/) enthusiast; Golang by itself deserves a
dedicated topic and **should be picked up by any DevOps who wants to ship
compact artifacts**.  But in order to build and write in Golang I need the Go
developer tool chain installed on my host machine to compile/build Go code...
right?

**No**.  You could write the code in your preferred environment.  When you are
ready to build, build with Docker.

{% highlight bash %}
# Find a place to make Gopher home
export GOPATH=/path/to/go
mkdir -p ${GOPATH}/src ${GOPATH}/pkg ${GOPATH}/bin
cd ${GOPATH}
# Setup script to run golang image as command
cat <<EOF >/usr/local/bin/go
#!/bin/bash
docker run --rm -v ${GOPATH}:/go golang:1.6 go \$@
EOF
chmod +x /usr/local/bin/go
# Now, lets get the official golang tutorial
go get golang.org/x/tour/gotour
# And run the tour
./bin/gotour -http 0.0.0.0:8080
{% endhighlight %}

{% include youtube.html id="ZGHlj5I1tFs" %}

### What is this mad magic
- We created a helper script to wrap the *docker command* to represent **go**
  binary
- This command creates and start a one time `container` using *golang:1.6*
  image.
- Bind directory `${GOPATH}` on your `host` to `/go` in the `container`.

### No like v1.6!!
Fine.  Change the image version to `golang:1.5`.

Think about what had just become available: **You can switch between language
versions without conflict with the one installed on your system**.

Imagine you are an author of a quite successful open source project.  You want
to support different versions of the language runtime.  To ensure that it works
over these environments, you need to build and test them properly.  Before, you
could try to spin up VMs to deal with these disparate environments, now you
simply build and test in different containers.

## A shippable environment
You might have picked up the trend here with the introduction of *Ruby* and
*Golang* base images.  These images are built and maintained by the
[Docker Hub](https://hub.docker.com/) team so that users could repliably
reproduce an environment with Ruby or Golang installed.  In the same vein, you
could build and ship full applications as Docker images and be confident that
these applications will not have unmet dependencies.

But a Docker image can be more then just an application package.  You could
ship **the whole environment** as an image, bells and whistles included, so that
people in your organization **works with consistent tool chain**.

Take my own [Dockerfile](https://github.com/jeffjen/workspace/blob/master/Dockerfile)
as an example, when I move to a new host machine, I can restore my workspace by

- Bootstrap the machine to run Docker
- Pull my workspace image and launch  
    - Golang, Node.js, Python, and Ruby installed
    - Command line utility tools installed
    - Text editor installed and configured
    - Shell environment installed and configured

## For the brave tinkerer
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
