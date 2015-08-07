---
layout: post
title: "Getting your Fingerprint Reader to Work"
date: 2015-08-06 22:46:35 -0500
comments: true
categories: laptops, readers, fingers
author: Alex Bird
---


If you have a Lenovo running Ubuntu with a fingerprint reader, it probably
doesn't work. Before you start downloading Windows 10, read this
guide on how to get the reader up and running. Note that I didn't come up with
any of this, I'm just presenting it nicely with appropriate witty remarks.

Power-up before you start this adventure:

```bash
bird@bird-840:~$ sudo -i
```

Then install some necessary packages:

```bash
# don't worry, you're not installing anything malicious...
root@bird-840:~# apt-get install -y fprintd fprint-demo libusb-1.0 libnss3 libnss3-dev libglib2.0-0 libglib2.0-dev libgdk-pixbuf2.0-0 libgdk-pixbuf2.0-dev imagemagick
```

Make sure you actually have a fingerprint reader:

```bash
root@bird-840:~# lsusb | grep Validity
Bus 002 Device 003: ID 138a:0017 Validity Sensors, Inc.
```

```bash
bird@bird-840:~$ git clone https://github.com/maffmeier/fprint_vfs5011.git fprint
# HEAD was 2af685532fa868cc76ef6acf17a94a0a1e8f3c20 as of this writing.
bird@bird-840:~$ cd fprint/libfprint/drivers
bird@bird-840:~$ vim vfs5011.c
```

Goto line 873 and add an entry in the `usb_id id_table[]` struct with the ID of
your _Validity Sensors, Inc._ device. For me it was this:

```c
// Bus 002 Device 003: ID 138a:0017 Validity Sensors, Inc.
{ .vendor = 0x138a, .product = 0x0017 /* vfs5011 */   },
```
Give yourself read permissions on the fingerprint reader USB device:

```bash
# '002' and '003' come from this, which hopefully looks familiar:
#
#     root@bird-840:~# lsusb | grep Validity
#     Bus 002 Device 003: ID 138a:0017 Validity Sensors, Inc.
#
$ sudo chmod +r /dev/bus/usb/002/003
$ sudo apt-get install glib-2.0 libmagickcore-dev
# verify
$ sudo fprint_demo
```




try 2
sources
- email
- http://tryitnw.blogspot.com/2013/02/easy-steps-to-enable-finger-print.html
```
$ sudo add-apt-repository ppa:fingerprint/fprint
$ sudo apt-get update
$ sudo apt-get install libpam-fprintd fprint-demo
$ ./configure && make && sudo  make install # does this overwrite of libfprint persist through package manager updates?
$ sudo fprint_demo
```

the hardest part is getting the swipe right
