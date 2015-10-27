---
title:  Optimizing for USB Drives
date:   2015-10-26 16:22:00
layout: post
tags: USB
---

We borrow hardware from various places on campus for our programming contest
every year. For a few years we used to wipe the machines, install our contest
environment, and then re-image them back to how they were when we were done.
Aside from being destructive to the data on the machines it is quite time
consuming.

For the past couple of years we've opted to run the contest off of usb flash
drives instead. Then all we have to change on the machines is the boot order and
we're done.

One problem that arises is some flash drives are not very fast. The USB2.0 spec
is only good for around ~60MB/sec. Typical performance you might see read speeds
in the tens of MB/sec. When a team tries to launch eclipse, firefox, java or any
other large program they are stuck waiting for it to read off the slow disk.

Now this isn't quite as bad as it sounds. The linux kernel is quite effective at
maintaining a disk cache in memory. Provided the system has sufficient ram,
we may only incur the read penalty from the usb flash drive once. Everything
after that will be read from the in-memory cache.

To improve things even further for our contestants, on boot we run a few commands
to pre-populate the kernel disk cache with various things teams may use. The first
thing we do is a walk the file system to cache some metadata about each file.
The second thing is to run a small tool called [vmtouch](http://hoytech.com/vmtouch/)
which will take a directory and read all of its contents into the disk cache. On
all systems we try to cache the following:

* The eclipse directory
* The java virtual machine
* C/C++ headers/includes
* A few of the scripts we use
* The contents of `/bin` and `/usr/bin` (Includes things such as firefox)

If the system has more than 2GB of memory, we also try to cache:

* `/lib` and `/usr/lib` - all the shared libraries various programs use

So on a decent system with say 4gb of ram we'll have already cached just about
everything a team might want to use. After this the speed of the usb flash
drive doesn't matter quite as much.

Since this does take a few minutes to run after booting we make sure all of the
machines are powered on and ready to go long before the teams sit down to use
them.

### Technical Details

For the technically inclined you'll want to take a look at the following files:

* [init script to run at boot](https://github.com/icpc-environment/icpc-env/blob/master/files/warm-fs-cache.conf)
* [vmtouch script](https://github.com/icpc-environment/icpc-env/blob/master/files/scripts/vmtouch.sh)
* [ansible playbook to configure everything](https://github.com/icpc-environment/icpc-env/blob/master/playbooks/vmtouch.yml)
