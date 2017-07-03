---
title: PCI passthru in bhyve
layout: post
date: 2017-07-03 03:08 EDT

---

I mentioned in an earlier post on virtualizing pfSense using bhyve that I would talk about PCI passthru.

The TL;DR of this is that, instead of using virtualized network cards or disk drives or whatever inside bhyve, we can actually "pass through" a PCI device so that a VM has exclusive and direct access to it.

This would have been incredibly useful for my pfSense purpose, because I could have had my firewall directly talk to the PCI ethernet port that my DSL modem is connected to rather than my dodgy **Ethernet -> Bridge -> virtual network (tap)** concoctation that theoretically I or a malicious person can accidentally expose my private networks too by assigning that bridge an IP or attaching another networking device to that bridge.

This _worked_ in the sense that I was able to get my pfSense VM seeing the PCI device. This _did not work_ in the sense that while a toy FreeBSD 11 VM was able to interact with the outside worldover that PCI device, for whatever reason the FreeBSD 10.4-based pfSense was not able to send traffic in a way that wasn't dropping packets all over the place.

Ah well.

There is a [post](https://murf.se/iohyve-and-pci-passthru/) out there that arguably has a succint instructio set for setting this up, but it's incredibly obtuse and explains nothing and is outdated now. This is the major reason why I ended up writing my own series of instructions in the first place. This is arguably just going to be an annotated version of the above blog post.

Glossary
==

**PCI**: Basically it's the physical standard for internal add-on computer components. We'll ignore that there's a external interface, and multiple successor interfaces also named PCI. Anyway, it's not an external device like USB.

**Bhyve**: The FreeBSD builtin virtualization software. It's pretty low-level and you would probably never use it directly.

**Iohyve**: The high-level use-friendly(ish) interface for bhyve that I chose to use.

**pfSense**: A software firewall I was trying to set up that was the impetus of this post, even though it really isn't relevant to this post now.

**Ethernet**: A networking standard. It's probably what you use for wired and wireless networking nowadays for regular consumer use.

How to set up device passthru in bhyve/iohyve.
=


Type `pciconf -lv`.

This will print a list of all PCI devices in your system. Find the device you want.

It'll look something like `em0@pci:10:0:0:    class=0x2000` ... in this case em0 indicates an Ethernet device.

If you knew you were looking for `em0` to being with, typing `pciconf -lv | grem em0` would have been more convenient.

**MEGA-NOTE:** If you have multiple devices like em0, em1, em2, and you make one a pass-through device (say em1), it will REORDER THE SUBSEQUENT DEVICES because the original em1 would have been taken out of the list of registered emX devices. Be warned, if you already have multiple devices of the same family set up.


Edit `/boot/loader.conf` to specify the PCI device that is being assigned as a passthru. Note that this only takes effect on a system start.

Your file should include a line like `pptdevs="10/0/0"`. Note how the number truplet mirrors what you saw above in em0@pci:**10:0:0:**.

You can totally pass through multiple devices, just add them in a space-separated list like `pptdevs=10/0/0 10/0/1 11/0/1"` etc....

**MINI-NOTE:** Some addon cards have multiple devices, which would show up as like two triplets, 9/0/0 and 9/0/1 or whatever. An example of this would be a network card with two or four thernet ports. Some devices are unhappy if you only pass through a subset of triplets ... I don't really understand it.

Let's create the pfSense (or whatever) virtual image you want to pass the device into....

`iohyve create pfsense 8G` (Create a VM with 8G hard drive space)


Let's assign the passthru device.

`iphyve set pfsense pcidev:7=passthru,10/0/0` (attach the passthru pci device. Note that it uses the triplets mentioned above. The triplets had to be mentioned in the loader.conf above. The **7** in `pcidev:7` is arbitrary and I don't know if there any any rules about what numbers you can and cannot use.


Enabling passthru devices in bhyve requires setting some startup flags...
`iohyve set pfsense bargs="-S_-A_-H_-P"`

The -S_ was added to the default `-A_-H_-P`. I think it stands for "wire guest memory", which means soemthing according to this [page](https://wiki.freebsd.org/bhyve/pci_passthru) that I don't really understand. You can remove it if you stop using PCI device passthru. Not setting it will make VMs with PCI passthru devices fail to start.


With this, you should see the device inside your virtual machine as if it was really attached to it. The host system no longer has the ability to use that device.
