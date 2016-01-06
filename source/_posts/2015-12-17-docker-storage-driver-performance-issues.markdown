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
tried-and-true, such as Ruby on Rails. But not always. Sometimes the
state of the art can deliver win after win, even amongst the inevitable trials
of using unproven software. Plus, the cutting edge is exciting and...actually,
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
time was due to the default Docker `--storage-driver` being different between
Ubuntu and CoreOS. Ubuntu was using `aufs`, and CoreOS `overlay`. With
`aufs`, it appeared that the parallel build processes were loading their
respective files in serial, each taking about 15 seconds. At 24 processes, the
6 minute slowdown is accounted for . With `overlay`, all 24 processes loaded
in the expected total time of ~15 seconds. After re-provisioning our Ubuntu
instances to use `--storage-driver=overlay`, we started to see expected
build times again. Huge relief.

This post wouldn't be complete without an example, so here we go.

### Setup

We'll need a multi-core instance to reproduce this behavior, so I spun up an
`m4.2xlarge` EC2 instance with the official Ubuntu 14.04 AMI, with a 50GB EBS
volume attached (called /dev/sdb). Let's get on the machine:

```
me@localhost$ ssh ubuntu@<instance_ip>
```

To run the `overlay` storage driver, we'll need at least the 3.18 Linux Kernel. Install:

```
ubuntu@ec2-instance$ sudo apt-get upgrade -y linux-generic-lts-vivid
ubuntu@ec2-instance$ sudo shutdown -r now
```

Get back on the train, and install Docker:

```
ubuntu@ec2-instance$ sudo -i

# paste all these lines at once:
apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
echo 'deb https://apt.dockerproject.org/repo ubuntu-trusty main' > /etc/apt/sources.list.d/docker.list
apt-get update -y
apt-get purge -y lxc-docker*
apt-get install -y docker-engine
usermod -a -G docker ubuntu
```

Log out and back in again to pick up the group change, and do some additional setup:

```
ubuntu@ec2-instance$ sudo -i

# paste all these lines at once:
service docker stop
mkfs.ext4 /dev/sdb
mount /dev/sdb /var/lib/docker
```

### aufs

Run the docker daemon:

```
ubuntu@ec2-instance$ sudo -i

# paste all these lines at once:
rm -rf /var/lib/docker
docker daemon -D --storage-driver=aufs
```

Instead of the Grand Rounds secret sauce, we'll be using
[discourse](https://github.com/discourse/discourse) as an example project. The
load time for its Ruby runtime is large enough to represent our problem well.

I've stuck all the discourse setup into a docker image.  Run the datastores
required for our test along with the discourse container:

```
ubuntu@ec2-instance$ docker run -d --name=redis --net=host redis:3.0
ubuntu@ec2-instance$ docker run -d --name=pg --net=host postgres:9.4
ubuntu@ec2-instance$ docker run -it --rm --name=storage-driver-test --net=host -w /root/discourse grnds/docker-storage-driver-test bash
root@ec2-instance:~/discourse# . ../setup.sh  # now we're in the container
```

We're ready to test our parallel problems, but let's get a baseline figure first:

```
# Ctrl-C the rspec command once it starts print dots, because we only care about the load time
root@ec2-instance:~/discourse# rspec
...^C
Finished in 6.68 seconds (files took 5.3 seconds to load)
...
```

The "files took 5.3 seconds to load" is our baseline. Let's run the `aufs` test:

```
# this will print out lots of stuff, and we don't care about most of it, but
# we can't kill the process early or the information we want won't get printed.
root@ec2-instance:~/discourse# parallel_rspec ./spec
8 processes for 330 specs, ~ 41 specs per process
...
Finished in 3 minutes 16.2 seconds (files took 25.16 seconds to load)
560 examples, 13 failures

Failed examples:
...
```

Eight rspec processes have run, and the "files took 25.16 seconds to load".
It's clearly not (baseline x 8), but it's much higher than expected. And
compared with `overlay`?

We'll have to re-do our setup, but docker ends up saving us quite a bit of
headache:

```
# kill the aufs docker daemon first.
# otherwise this is exactly the same as above, other than the storage driver.
ubuntu@ec2-instance$ sudo -i
root@ec2-instance# rm -rf /var/lib/docker
root@ec2-instance# docker daemon -D --storage-driver=overlay

ubuntu@ec2-instance$ docker run -d --name=redis --net=host redis:3.0
ubuntu@ec2-instance$ docker run -d --name=pg --net=host postgres:9.4
ubuntu@ec2-instance$ docker run -it --rm --name=storage-driver-test --net=host -w /root/discourse grnds/docker-storage-driver-test bash
root@ec2-instance:~/discourse# . ../setup.sh  # now we're in the container

root@ec2-instance:~/discourse# parallel_rspec ./spec
8 processes for 330 specs, ~ 41 specs per process
...
Finished in 1 minute 6.27 seconds (files took 6.04 seconds to load)
592 examples, 21 failures

Failed examples:
...
```

Here, the eight rspec processes took 6.04 seconds to load their files. It's
very close to the baseline.

This analysis certainly has confounders, but it captures the pattern of poor
performance we were seeing in production on nearly the same stack and in an
easily reproducable way.

I don't think we necessarily learned any big lesson from this. I think it
mainly served to reinforce that you can't plan for everything when you make big
changes, and that an in-depth knowledge of your system and good troubleshooting
skills are the only things that can save you from yourself.
