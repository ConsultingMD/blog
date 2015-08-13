---
layout: post
title: "Getting your Fingerprint Reader to Work"
date: 2015-08-06 22:46:35 -0500
comments: true
categories: laptops, readers, fingers
author: Alex Bird
---
If you are running Ubuntu on a laptop with a fingerprint reader, it probably
doesn't work.  Before you start downloading Windows 10, read this guide on how
to get the reader up and running. I'm using a Lenovo T450S, but this may work for
other manufacturers and models.

First we're going to be installing
[fprint](http://www.freedesktop.org/wiki/Software/fprint/), a library which
attempts to unify as many hardware fingerprint readers as possible under a
common interface. That will include some tools for scanning our fingers into
the system.

We'll also install the fprint module into Linux's [Pluggable Authentication
Modules (PAM)](https://en.wikipedia.org/wiki/Linux_PAM) system.  This will
allow seamless integration with the standard linux authentication systems, like
login.

Then we'll realize our fingerprint reader isn't covered by fprint, so we'll
add it to the fprint source, compile it, and install it over the apt-get one. It ain't
pretty, but it's simple enough and it will get things working. This will enable us to
add our fingerprints to the system and verify that everything is working.

Finally, we'll realize that the hardest part of all this is getting the reader
to recognize your fingerprints.

## Do you have a Fingerprint Reader?
Check for a fingerprint reader device on your system:
```
$ lsusb
Bus 001 Device 002: ID 8087:8001 Intel Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 004: ID 17ef:1010 Lenovo
Bus 003 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 002 Device 005: ID 04f2:b449 Chicony Electronics Co., Ltd
Bus 002 Device 004: ID 8087:0a2a Intel Corp.
Bus 002 Device 003: ID 138a:0017 Validity Sensors, Inc.           # yes!
Bus 002 Device 002: ID 04f3:0418 Elan Microelectronics Corp.
Bus 002 Device 017: ID 062a:4102 Creative Labs
Bus 002 Device 016: ID 17ef:100f Lenovo
Bus 002 Device 015: ID 058f:9410 Alcor Micro Corp. Keyboard
Bus 002 Device 014: ID 17ef:1010 Lenovo
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
Validity Sensors, Inc. is the vendor for mine, so we can proceed.

## Install fprint & PAM Module
This step is pretty simple:
```bash
sudo add-apt-repository ppa:fingerprint/fprint
sudo apt-get update
sudo apt-get install libpam-fprintd fprint-demo
```
Make sure the PAM module was installed:
```bash
$ grep fprint /etc/pam.d/common-auth
# you should see output like this:
auth [success=2 default=ignore] pam_fprintd.so
```
At this point you might be tempted to it try out, but running `sudo
fprint_demo` will probably show you that no device could be found. Fine.

## Custom fprint Build
Get the source, which in this case is specific to the reader in my machine:
```bash
# HEAD was 2af685532fa868cc76ef6acf17a94a0a1e8f3c20 as of this writing.
git clone https://github.com/maffmeier/fprint_vfs5011.git fprint
cd !$
vim libfprint/drivers/vfs5011.c
```
Go to line 873 and add an entry in the `usb_id id_table[]` struct with the ID of
your _Validity Sensors, Inc._ device. For me it was this:
```c
// Bus 002 Device 003: ID 138a:0017 Validity Sensors, Inc.
{ .vendor = 0x138a, .product = 0x0017 /* vfs5011 */   },
```
Install some build dependencies and run the build:
```bash
sudo apt-get install glib-2.0 libmagickcore-dev
./configure && make && sudo make install
```
Give yourself read permissions on the fingerprint reader USB device:
```bash
# '002' and '003' come from this, which hopefully looks familiar:
#
#     $ lsusb | grep Validity
#     Bus 002 Device 003: ID 138a:0017 Validity Sensors, Inc.
#
sudo chmod +r /dev/bus/usb/002/003
# verify
sudo fprint_demo
```
You should be able to scan your fingerprints now.

## Using the Technology
Unfortunately, the hardest part of all this seems to be consistently swiping
your finger in exactly the right way such that it gets read. A couple of things
that have I have noticed for successful fingerprint reading are:
- swipe at a constant speed
- move your finger fast but not too fast
- angle your fingerprint directly over the scanner

I've seen different "successful scan" behaviors, depending on the lock screen
in use. For the default lock screen, the GUI will tell you when it wants you to
scan, and when there is a failure, but it won't change at all for a good scan.
And you have to press enter when you notice that nothing has changed. For
`gnome-screensaver-command --lock`, however, the screen will unlock as soon as
the scan is good. I haven't been able to get `xscreensaver-command -lock` to
work yet. Good luck!

##### Sources
- Kevin Moore, Ken Berland of the Grand Rounds Engineering Team
- [http://tryitnw.blogspot.com/2013/02/easy-steps-to-enable-finger-print.html](http://tryitnw.blogspot.com/2013/02/easy-steps-to-enable-finger-print.html)
