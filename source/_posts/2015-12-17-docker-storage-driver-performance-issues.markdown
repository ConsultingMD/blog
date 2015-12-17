---
layout: post
title: "Docker Storage Driver Performance Issues"
date: 2015-12-17 12:43:40 -0800
comments: true
author: Alex Bird
categories: docker, coreos, ubuntu, aufs, overlayfs
---

Here at Grand Rounds, we're pragmatists when it comes to choosing the
technologies we work with on a daily basis. Often, that means using what's
tried and true, such as Ruby on Rails. But not always. Sometimes the
state-of-the-art can deliver win after win, even amongst the inevitable trials
of using unproven software. Plus, the cutting-edge is exciting and...actually,
screw that. Stable systems are better than any of that nonsense.

"Honesty", our custom CI system, was originally built on CoreOS. A year and
many trials later, we've moved completely off the CoreOS ecosystem. We're still
using Docker, but now we're on the very familiar Ubuntu, and using Docker Swarm
for clustering. To document all of the weird problems we worked around or never
quite got a handle on, while keeping the team's builds moving along smoothly
would be a sort of fishing tale: of interest mainly to those who were there,
and pretty boring otherwise.

One struggle that popped up recently may be helpful to document though. After
flipping the switch on our new Ubuntu instances, we started seeing
significantly slower build times, on the order of an additional five minutes.
Digging in, we found that the slowdown was coming from our parallelized Ruby
build processes starting up. Eventually, we figured out that the additional
time was due to the default Docker storage driver being different between
Ubuntu and CoreOS. Ubuntu was using `aufs`, and CoreOS `overlayfs`. With
`aufs`, it appeared that the parallel build processes were loading their
respective files in serial, each taking about 15 seconds. At 24 processes, the
6 minute slowdown is accounted for . With `overlayfs`, all 24 processes loaded
in the expected total time of ~15 seconds. After re-provisioning our Ubuntu
instances with `overlayfs` as the storage driver , we started to see expected
build times again. Huge relief.

I'm not sure we learned any big lesson from this. I think it mainly served to
reinforce that you can't plan for everything when you make big changes, and
that an in-depth knowledge of your system and good troubleshooting are the only
things that can save you from yourself.
