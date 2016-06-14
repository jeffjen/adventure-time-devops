---
layout: post
title: "Thinking in Micro Services"
date: 2016-06-12 10:00:00 +0000
categories: docker
tag:
description: >
excerpt_separator: <!--more-->
---

In system architecture 101 you are constantly warned about the importance of
service high availability, and time again you see designs of system are
separated at the most basic form:  
- Keep data and business logic separate
- Load balance your grunts, service workers

Then people begin to realize that service hit rate are not uniform:
certain services have higher request density then others; heterogeneous roles
amongst your services require conflicting optimization goals: CPU, Disk, or
Memory.  This is the signature problem of a **Monolithic Application**
architecture.  While troublesome, people ran with it anyway.  This is because
the alternative, that rising star **Micro Service Application** is simple to
utter, but difficult to manage.

So, what is a **Micro Service Application**?  I will try to explain this with
less fervor and more with grounded expectation so you as an architect have
better expectation.

Also, there will be alot of big words jotting out here and there in the article
so *brace yourselves*.

<!--more-->

## Micro Service Application is hard
Micro Service Application bores out from the necessity to build a lighter and
more agile service from Monolithic Application.  In essense, instead of
bundling everything you want to run on a single machine, partition it into
logical *service group*(or in fancy terms: Service Oritented Architecture).
The goal is to optimize for different work loads and patterns:  
- Static asset server.
- API server
- Worker
- Datastore

However, the senisible solution here to group service into resource optimised
for their work pattern creates a side effect that many would walk away from.
These are:  
- Service discovery
- Scheduling

### Service discovery
The term service discovery refers to how services find out each other.  The
most basic service discovery is locating the datastore.  In advacned usage, you
would use register API for different purposes and identifiy them as a services
e.g. An API that spawns worker to deal with batch job could be
at *batch.example.org*

### Scheduling
Work loads and tasks you want to place them where their required execution
resource is met.  A scheduler is created to manage these requirements without
adminstrator's intervention.

If I may make a assertive statement here, I would say that Micro Service
Application is by no means an easier solution then Monolithic Application.  In
fact in all aspects the former is an advance construct from Monolithic and is
natrually harder to solve then the latter.  Bear in mind that most organization
roll with Monolithic Application.  This is not necessarily because people are
lazy, but it is sufficient for their business.

## Plan your infrastructure to deal with upgrade overtime
To tackle such a gargantuan task one must sort out the priorities.  As with
many things in life it comes down to the level of preparedness you are
comfortable living with. A perfact system that address all needs does not
exist.  A system, unlike a building or bridge, is a living being.  New
requirements are added every day, and changes will come not just from you, but
also from dependencies.
