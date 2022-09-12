---
layout: home
title: Introduction
nav_order: 0
---

# Two tauers: the docs, the guide, the cat tax

![Homepage image with a cat](../assets/images/home.jpg "two tauers and the tax")

Two tauers is a set of tools for setting up a k3s cluster on Raspberry Pi's.
The core aim is to make this process as repeatable as possible with ansible and helm.

As there are several more or less distinct steps, they each have their own repo in [two-tauers](https://github.com/two-tauers) organisation on github.
"Organisation" is a fancy word, but currently it is maintained by one person who used this as a project to learn about kubernetes and ansible.

The name "two tauers" was coined by my colleague who introduced me to the value of &tau; = 2&pi;.

### Scope

So far there is a [set of ansible roles](https://github.com/two-tauers/baseline) that set up SSH access for a freshly flashed Ubuntu Server and prepare it for k3s installation (that is done using a [fork of the official k3s-ansible repo](https://github.com/two-tauers/k3s-ansible)).
Persistent storage and observability is in the works -- this will be the bare minimum for a cluster setup.

In addition, when the above is implemented, tested and documented, there are plans for setting up ingress for web services and running the whole thing on Alpine Linux.

### Use cases

This setup can be used as a home server, running Home Assistant, Pihole, NAS, Github Actions etc.
However, that is not the end goal of this project, that is up to whoever is setting it up.

### Why k3s on Raspberry Pi's

There are many valid arguments against using Raspberry Pi's for anything apart from the fun of learning -- low power, terrible support for ARM architecture, the fact that with a little more money you can get quite a bit beefier mini PC etc.
However, there are some benefits as well.
They small, quiet, and power efficient.
Plus a Raspberry Pi clsuter can be a functional kinetic sculpture on your desk if you're into that.
