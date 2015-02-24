---
layout: post
title: "Running Local CoreOS Workers"
date: 2015-02-23 22:49:10 -0800
comments: true
categories: coreos
author: Ken Berland <ken@grandroundshealth.com>
---
We're working with CoreOS here, both in the cloud and locally.  For 24/7 workloads, local servers are far
more cost effective than "big iron"-type instances in the cloud, like c3.8xlarge or r3.8xlarge.  These can be around
$2/hour, so running locally for things like continuous integration can pay back in a month or less.

Here's how to boot a local worker using a CoreOS USB image and have it upgrade automatically.

### Make a bootable USB from the stock CoreOS image.

- Insert your USB stick and make note of the device, here we're using `/dev/sdb`.
- Download the stable CoreOS .iso.  Later, in you cloud-config, you can choose from stable, beta, or alpha.
```
wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_iso_image.iso
```

- Your cloud-config that we'll create below will be available on the web.  Create a `syslinux.cfg` that knows where it is:

```
cat <<EOF > syslinux.cfg
default coreos
prompt 1
timeout 15

label coreos
  menu default
  kernel /coreos/vmlinuz
  append initrd=/coreos/cpio.gz coreos.autologin cloud-config-url=http://your.domain.com/cloud-config.yml
EOF
```

- Now, we'll mount the stock image, format your USB, copy the stock image, and make it bootable with syslinux.
```
DEVICE=/dev/sdb
mkdir stock
sudo mount -oro,loop coreos_production_iso_image.iso ./stock
dd if=/dev/zero of=${DEVICE} bs=512 count=1
fdisk $DEVICE <<EOF
n
p
1

+2048M
t
6
a
1
w

EOF

sudo mkfs.msdos -F 16 ${DEVICE}1
mkdir vfat
sudo mount ${DEVICE}1 ./vfat/
cd vfat/
sudo cp -r ../stock/* .
sudo cp ../syslinux.cfg syslinux
cd ..
sudo umount vfat/
syslinux --install ${DEVICE}1 --directory syslinux
dd if=./stock/isolinux/mbr.bin of=${DEVICE} # see man syslinux and search for mbr
sync # you can never be too careful
```

### Create a cloud-config that Auto-Updates

Below, we've listed the complete cloud-config.yml.  Here is a bit of explanation of the individual pieces.

- you'll want your local instance to use local storage.  So, add services that format
the local disk as btrfs and labels it so Docker can find it:

```yaml
  - name: format-ephemeral.service
    command: start
    content: |
      [Unit]
      Description=Formats the ephemeral drive
      Before=var-lib-docker.mount
      [Service]
      Type=oneshot
      RemainAfterExit=yes
      ExecStart=/usr/sbin/wipefs -f /dev/sda
      ExecStart=/usr/sbin/mkfs.btrfs -LDOCKER -f /dev/sda
  - name: var-lib-docker.mount
    command: start
    content: |
      [Unit]
      Description=Mount ephemeral to /var/lib/docker
      Before=docker.service
      [Mount]
      What=LABEL=DOCKER
      Where=/var/lib/docker
      Type=btrfs
```

- The stock CoreOS image has a file called `/usr/.noupdate` that interacts with several systemd service files
and stops the instance from auto-updating.  The culprit looks like this:
```
ConditionPathExists=!/usr/.noupdate
```

- Rewrite those services without the `ConditionPathExists`:

```yaml
  - name: locksmithd.service
    command: start
    content: |
      [Unit]
      Description=Cluster reboot manager
      Requires=update-engine.service
      After=update-engine.service
      ConditionVirtualization=!container
      [Service]
      CPUShares=16
      MemoryLimit=32M
      PrivateDevices=true
      EnvironmentFile=-/usr/share/coreos/update.conf
      EnvironmentFile=-/etc/coreos/update.conf
      ExecStart=/usr/lib/locksmith/locksmithd
      Restart=always
      RestartSec=10s
      [Install]
      WantedBy=multi-user.target
  - name: update-engine.service
    command: start
    content: |
      [Unit]
      Description=Update Engine
      ConditionVirtualization=!container
      [Service]
      Type=dbus
      BusName=com.coreos.update1
      ExecStart=/usr/sbin/update_engine -foreground -logtostderr -no_connection_manager
      BlockIOWeight=100
      Restart=always
      RestartSec=30
      [Install]
      WantedBy=default.target
```

- Pick your channel:

```yaml
coreos:
  update:
    reboot-strategy: best-effort
    group: alpha
```

- You might want to schedule your reboots for a specific time of day when no one is likely to notice.  Add some services to do that:

```yaml
  - name: update-window.service
    content: |
      [Unit]
      Description=Reboot if an update has been downloaded
      [Service]
      ExecStart=/usr/bin/bash -c 'if update_engine_client -status | grep NEED_REBOOT; then reboot; fi'
  - name: update-window.timer
    command: start
    content: |
      [Unit]
      Description=Reboot if needed at 05:00 daily
      [Timer]
      OnCalendar=*-*-* 05:00:00
```

Here's the complete cloud-config.  Remember to add some ssh public keys so that you can login easily.  And don't forget the pesky
comment at the top of the file.

```yaml
#cloud-config
---
coreos:
  fleet:
    etcd_servers: http://etcdcluster1.grandrounds.com:4001, http://etcdcluster2.grandrounds.com:4001,
      http://etcdcluster3.grandrounds.com:4001
    metadata: role=worker
  update:
    reboot-strategy: best-effort
    group: alpha
  units:
  - name: locksmithd.service
    command: start
    content: |
      [Unit]
      Description=Cluster reboot manager
      Requires=update-engine.service
      After=update-engine.service
      ConditionVirtualization=!container
      [Service]
      CPUShares=16
      MemoryLimit=32M
      PrivateDevices=true
      EnvironmentFile=-/usr/share/coreos/update.conf
      EnvironmentFile=-/etc/coreos/update.conf
      ExecStart=/usr/lib/locksmith/locksmithd
      Restart=always
      RestartSec=10s
      [Install]
      WantedBy=multi-user.target
  - name: update-engine.service
    command: start
    content: |
      [Unit]
      Description=Update Engine
      ConditionVirtualization=!container
      [Service]
      Type=dbus
      BusName=com.coreos.update1
      ExecStart=/usr/sbin/update_engine -foreground -logtostderr -no_connection_manager
      BlockIOWeight=100
      Restart=always
      RestartSec=30
      [Install]
      WantedBy=default.target
  - name: fleet.service
    command: start
  - name: update-window.service
    content: |
      [Unit]
      Description=Reboot if an update has been downloaded
      [Service]
      ExecStart=/usr/bin/bash -c 'if update_engine_client -status | grep NEED_REBOOT; then reboot; fi'
  - name: update-window.timer
    command: start
    content: |
      [Unit]
      Description=Reboot if needed at 05:00 daily
      [Timer]
      OnCalendar=*-*-* 05:00:00
  - name: format-ephemeral.service
    command: start
    content: |
      [Unit]
      Description=Formats the ephemeral drive
      Before=var-lib-docker.mount
      [Service]
      Type=oneshot
      RemainAfterExit=yes
      ExecStart=/usr/sbin/wipefs -f /dev/sda
      ExecStart=/usr/sbin/mkfs.btrfs -LDOCKER -f /dev/sda
  - name: var-lib-docker.mount
    command: start
    content: |
      [Unit]
      Description=Mount ephemeral to /var/lib/docker
      Before=docker.service
      [Mount]
      What=LABEL=DOCKER
      Where=/var/lib/docker
      Type=btrfs
ssh_authorized_keys:
- |
  ssh-rsa AAAAB3NAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA== someone@grandrounds.com
- |
  ssh-rsa AAAAB3NAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA== another@grandrounds.com
```

### Done!

Now you've got a powerful CoreOS worker locally for a fraction of the 24/7 price at AWS or another cloud provider.  Enjoy.