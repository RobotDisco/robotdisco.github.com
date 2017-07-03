---
title: Assorted notes on getting FreeBSD jails running with VLANs and my virtual pfSense firewall
layout: post
date: 2017-07-03 02:17 EDT

---

*Note: This post assumes you know how VLANs work, I am not capable of explaining them succinctly and there are good resources out there*


Jails are interesting. They are _not_ virtual machines, they're in a sense glorified chroots with some process isolation around them. I feel like everyone tries to say this but I've never read a succinct explanation as to what they provide and what they don't.

Jails share the host's network infrastructure. Since jails run as root by default, I am unsure why a jail cannot simply add itself to an exposed ethernet connection of the host, since I can _see_ the configured devices. There must be some security aspect of jails I do not yet understand.

In theory I can recompile the FreeBSD kernel with VIMAGE/VNET support (see https://jsherz.com/freebsd/jails/kernel/vnet/2015/11/17/custom-kernel-jails-vnet.html) to give each jail its own independant network stack. Perhaps this would be a good idea for running pfSense in a jail, but I never got it working for other jailed software.

There seems to be some interaction issue between VNET/VIMAGE, vlans, and bridges. Because my FreeBSD vlan device spawns off the _bridge_, not the physical device I'm using to connect pfSense with my switch. But jails need both the parent and vlan pseudo-ethernet device attached to the jail bridge, and that defeats the point of tagged VLANs AFAICT. It seemed like a dead end, and my jails would crash the system because of some bug in the pre-release VNET driver or something.

What ended up saving me was learning that FreeBSD has _multiple_ routing table support. My host machine uses a routing table for the administration VLAN, but I can set up a second one for my server VLAN and a third one for my exposed-to-big-bad-internet VLAN. I used the lifesaver post at https://savagedlight.me/2014/03/07/freebsd-jail-host-with-multiple-local-networks/.

As of this writing, iocage has a bug in its binary package that doesn't respect the routing table number you ask the jail to use. This is fixed, happily, in the source and installing iocage via the ports system works correctly.