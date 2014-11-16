---
layout: post
title: "Compile a Kernel in Ubuntu"
date: 2014-11-16 12:16:58 -0800
comments: true
categories:
---
I often get razzed by OSX users for compiling a kernel.  Suffice to say, I don't understand it as a complaint or a put-down.  Controlling your kernel is an important part of understanding the stack.  And running the same OS in production as on your desktop can teach you lessons that later save you time (or save your ass).

Anyway, compiling a new kernel is pretty simple in ubuntu:

```bash
# copy your current config to the new kernel's working dir
# download the latest kernel from kernel.org
export VERSION=3.17.3 # e.g.
cd
wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-$VERSION.tar.xz
cd /usr/src
tar -xf ~/linux-$VERSION.tar.xz
ln -s linux-$VERSION linux # make /usr/src/linux a link to the source
cd linux
cp /boot/config-`uname -r` .config  # get your current config
yes | make oldconfig # hit 'Y' a million times or pipe in the output from `yes`
make  # make the kernel
make modules
make headers_install
sudo INSTALL_MOD_STRIP=1 make modules_install # remove debugging symbols for smaller /lib/mmodules and initramfs
cd /boot
sudo cp /usr/src/linux/arch/x86/boot/bzImage vmlinuz-$VERSION
sudo cp /usr/src/linux/System.map System.map-$VERSION
sudo cp /usr/src/linux/.config config-$VERSION
sudo mkinitramfs $VERSION -o ./initrd.img-$VERSION
sudo grub-mkconfig > /tmp/grub.cfg
sudo cp /tmp/grub.cfg /boot/grub/grub.cfg
# reboot!
```
