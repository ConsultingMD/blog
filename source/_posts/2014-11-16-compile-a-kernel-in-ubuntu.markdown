---
layout: post
title: "Compile a Kernel in Ubuntu"
date: 2014-11-16 12:16:58 -0800
comments: true
categories:
author: "Ken Berland <ken@grnds.com>"
---
I often get razzed by OSX users for compiling a kernel.  Suffice to say, I don't understand it as a complaint or a put-down.  Controlling your kernel is an important part of understanding the stack.  And running the same OS in production as on your desktop can teach you lessons that later save you time (or save your ass).

Anyway, compiling a new kernel is pretty simple in ubuntu:

- Download the latest kernel from kernel.org and unpack it:

```bash
export VERSION=3.17.3 # e.g.
cd
wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-$VERSION.tar.xz
cd /usr/src
tar -xf ~/linux-$VERSION.tar.xz
ln -s linux-$VERSION linux # make /usr/src/linux a link to the source
cd linux
```

- The kernel is configured in one file containing all the options.  Ubuntu stores it in `/boot`.  Get a copy of it and `oldconfig` to configure all the new options as default:

```bash
cp /boot/config-`uname -r` .config  # get your current config
yes | make oldconfig # hit 'Y' a million times or pipe in the output from `yes`
```

- Compile the kernel.  Set -j to the number of cores you have.  This will take some time:

```bash
make -j8 # make the kernel
make -j8 modules
```

- Install the kernel headers and the loadable modules:

```bash
make headers_install
sudo INSTALL_MOD_STRIP=1 make modules_install # remove debugging symbols for smaller /lib/mmodules and initramfs
```
- Now install the kernel into `/boot`:
```bash
cd /boot
sudo cp /usr/src/linux/arch/x86/boot/bzImage vmlinuz-$VERSION
sudo cp /usr/src/linux/System.map System.map-$VERSION
sudo cp /usr/src/linux/.config config-$VERSION
```

- Make an initial disk to boot the machine that the kernel can use before the filesystem is available:

```bash
sudo mkinitramfs $VERSION -o ./initrd.img-$VERSION
```

- Make a new Grub configuration
```bash
sudo grub-mkconfig > /tmp/grub.cfg
sudo cp /tmp/grub.cfg /boot/grub/grub.cfg
```

- Reboot with your new kernel:

```bash
reboot
```
