---
layout: post
title: "Golang is the best command line language"
date: 2016-06-26 10:00:00 +0000
categories: aws docker ecs golang lambda
tag:
  - aws
  - docker
  - ecs
  - golang
  - lambda
description: >
  Developing for customers running on different platforms is an ordeal.
  Fear not, there is **Golang**.
excerpt_separator: <!--more-->
---

Developing for customers running on different platforms is an ordeal.
Scripting language such as *Python*, *Node.JS* provides some level of
"cross-platform" which provides a consistent environment for executing the
same program through the support from the language runtime.  However, even with
the hardwork and marvelous support from the runtime, package written with
native C/C++ is not going to waltz into a different Linux distro, let alone
onto Mac or Windows systems.

The daunting task you have to overcome to get it just right is a constant dread
that makes every developer pump their fists in the air crying in silent grief:
"Why oh why these people couldn't just run with the environment I run with!"

Fear not, there is **Golang**.

<!--more-->

Golang is a strongly typed, compiled language where in many ways look like the
*C* programming language.  Where its different is in the following areas:

- Cross platform build capabilities.
- Functional programming without caveats.
- Natvie support for concurrent execution.

Let us talk about
[Cross platform build capabilities](#cross-platform-build-capabilities), and
move on to [Functional programming and Cuncurrent design]()

## Cross platform build capabilities
Before you run a program, say Python or Node.JS, you need to first make sure
that the runtime for the respective language is installed.  This is because
Python and Node.JS script is not understood by the operating system: it is the
runtime's responsibility to parse the *program* into sensible form to execute,
hence the term **Interpreted Language**.

The problem with this approach is apparant:

- You need the runtime installed
- You need to make sure that dependencies will also run on the target
  environment.

Say you are developing with Python.  If you develop in a later version of
*Python 3.x*, but your client is running *Python 2.x*, you cannot simply tell
them to ditch their runtime for another, as both their system infrastructure
depends on *Python 2.x*.

You could ship the program in [Docker](https://www.docker.com/), but
lets also supoose your client is running Mac or Windows.  **Docker could
only run on Linux**.  At this point you would already be cursing your client's
antiquated and bizarre setup.

If you had developed in **Golang**, the only thing you need to do to ship to
Linux, Max, Windows, Rasperry Pi is with an environment toggle:

{% highlight bash %}
GOOS=darwin go build
{% endhighlight %}

You had just build a binary to run in Mac environment.  If you are on a
Linux/Mac machine, you can verify this is true by running
`file [your binary name]`.  It will report:

{% highlight bash %}
[program name]: Mach-O 64-bit x86_64 executable
{% endhighlight %}

Here is a [list](https://github.com/golang/go/blob/master/src/go/build/syslist.go)
of available platforms to build into, most notably:

- linux
- darwin
- windows
- arm

And well known architectures:

- 386
- amd64
- arm
- arm64

To further clamp down on variables in the shipped product, you could build the
program through static linking, removing the need to ship libraries.

{% highlight bash %}
GOOS=linux CGO_ENABLED=0 go build -a -installsuffix cgo
{% endhighlight %}

There you go.  You can be confident that the program you ship will be
immediatly available to you customer without installing any runtime.

## A real life example
Let me demonstrate to you why the ability to run a program without a runtime
is great power.

As a DevOps engineer I have dabbled in the use of
[AWS Lambda](http://docs.aws.amazon.com/lambda/) and integrating it here and
there to automate infrasturcture workflow.

Being a [Docker](https://www.docker.com/) user, all of my provisioned EC2
instances are natrually pre-installed with Docker.  In addition,
[ECS](https://aws.amazon.com/ecs/) provides interesting ways to schedule one
off tasks, but needs registering to ECS.  Finally, I use
[Weave](www.weave.works) to provide service discovery.

A few problems pop into mind:

- How do I install ECS agent, if the agent should register to different
  clusters?
- How do I install `Weave` into different network?
- How do I make `Docker` Host available to me via secure connection?

What is especially difficult is getting the **Docker Host TLS** setup
correctly, using **SSH remote command** to configure the system.  This is the
single most important step since we cannot let unaurthorized request to our EC2
instances, but retain the ability to launch containers remotely.

Writing a complex program such as this from the ground up is difficult,
whereas writing in shell script is simple.  Fortunately for us, `Docker` Client
is written in **Golang**, statically linked and availble for any Linux
environment.  `Weave` provides a
[helper shell script](https://www.weave.works/install-weave-net/) to join a
`Docker` Node to a network, which also uses `Docker` Client.

The missing link to provision the instance as a `Docker` Node and TLS ready is
through the use of [Machine](https://github.com/jeffjen/machine), which also is
written in **Golang**.  `Machine` provides:

- DNS SRV record look up
- Easy TLS certification generation
- Configuring `Docker` Host to run over TLS

Tying all of this together is a Node.JS Lambda handler
[ecs-docker-engine](https://github.com/jeffjen/lambda-ecs-docker-engine).
Which is a simple handler to respond to events published to
[AWS SNS](https://aws.amazon.com/sns/) by
[AWS AutoScaling](https://aws.amazon.com/autoscaling/).  The core of the
handler is a shell script fork and exec based on the event received.

Here is a snippet of the shell script:
{% highlight bash %}
# Configure Docker Engine for remote access
machine --certpath=${CACERT} --skip-instance-cache=true tls gen-cert-install --host=${PUBLIC_IP} --altname=${PRIVATE_IP}

# Setup Docker Client environment
export DOCKER_HOST=tcp://${PUBLIC_IP}:2376
export DOCKER_TLS_VERIFY=1
export DOCKER_CERT_PATH=${PWD}/cert

# Instruct weave to install router and network plugin
weavedb=$(docker ps -a --format '{{ .Names }}' -f name=weavedb)
if [ -z ${weavedb} ]; then
weave launch $(machine dns lookup-srv ${SRV} ${ZONE})
fi

# Configure target machine to join an ECS cluster
# See http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-agent-install.html
agent=$(docker ps -a --format '{{ .Names }}' -f name=ecs-agent)
if [ -z ${agent} ]; then
docker run --restart=on-failure:10 --net host --name ecs-agent -d \
    -v /var/log/ecs/:/log \
    -v /var/lib/ecs/data:/data \
    -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
    -e DOCKER_HOST=tcp://localhost:12375 \
    -e ECS_LOGFILE=/log/ecs-agent.log \
    -e ECS_LOGLEVEL=info \
    -e ECS_DATADIR=/data \
    -e ECS_CLUSTER=${ECS_CLUSTER} \
    amazon/amazon-ecs-agent:latest
fi
{% endhighlight %}

All of this shell script munging can be written in Node.JS.  But getting it
right is difficult.  `Docker` and `Machine` provides these facilities
and tested to death, so leveraging them is sensible when latency is not a
huge issue.

All made possible because of **Golang**.

## Moving on to the language it self
If the prospect of the cross-platform capabilities piques your interest in
using **Golang** you will need to learn more about the langauge itself.  Read
on the basic principles in [Functional programming and Cuncurrent design]().
