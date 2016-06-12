---
title: SmartOS will hang if you try to install it using Intel AMT's remote desktop tool!
layout: post
date: 2016-06-12 15:11 EDT

tags: smartos, intel amt, server administration
---

This is primarily a note for anyone who, like me, was hoping to install SmartOS on a server using nothing but Intel's AMT technology.

Definitions
===========

For those wondering,

* SmartOS is a derivative of Illumos a.k.a. the Open Source version of Solaris, forked from before Oracle purchased Sun Microsoystems and made the codebase proprietary again. It's optimized for "hypervisors", a.k.a. having a single large machine running many small virtual machines instead.
* Intel AMT is a out-of-band server management technology. Basically, it means you can manage a hardware server (or many hardware servers at a time!) without needing to directly plug a mouse or keyboard or monitor in. Instead, there's some magic that allows the ethernet port to also listen to some kind of remote management protocol even while the network card is being used for regularly network things! It's very handy for people in datacentres who have to manage many machines! It's also handy for lazy hobbyists like myself who don't want to lug a monitor or keyboard downstairs every time I mess something up :)
* KVM in _this_ context means software that allows you to pretend to use a server's keyboard/video/mouse when in fact you're connecting to it remotely from another machine. A feature of newer Intel AMT versions is that the server has an embedded KVM functionality built in, so _from the moment the server boots_ I can control it remotely, including hardware settings. That's so cool! I wish all computers had this :)

The TL;DR
===========
This post is going to say that if you're trying to use the KVM functionality on a server to install SmartOS, the install will mysteriously crash.

I noticed this because during the rescue mode everything would work fine. I eventually traced through the SmartOS install script and isolated the command that caused everything to crash.

The reason is because the install process initializes each network interface using `dladmin plumb`... since one of them is the one you are connecting to via Intel AMT's keyboard/mouse/video-over-ethernet connection, the KVM connection will reset and it will seem like your machine froze :(

The smart thing to do would to cut your losses and bring down a monitor and a keyboard. Once SmartOS is set up, this stops mattering since you can connect afterwards via SSH or via the KVM if you really wanted to.

The _fun_ thing to do, which I did because SmartOS uses a commandline stack completely foreign to me and I learn best by diving in, is to read through all the install scripts and set up everything manually :)

Other speculations
=====

I happened to buy a Dell PowerEdge T20 server, which is a lower-end server designed for home offices. I know that the higher-end Dell servers, build-it-yoursself motherboards from SuperMicro, and probably others use a different remote management technology (IPMI?) which I know nothing about but people seem to prefer it.
Maybe for non-hobbyist data centres that is true because you can have some fancy server to control countless machines from one nice interface, but for lil' hobbyist me all I need is a machine that I don't need to connect a monitor or keyboard too, since my server is stuck in a corner somewhere and nowhere near to my home workstation.

I also notice that most higher-end servers have multiple network cards, one card is specifically used for remote administration and the others for network use. This probably makes sense in a data centre where you can physically separate your networks for security and isolation, but I imagine it'd be overkill for me. I'm not really sure what the ramifications of this are yet.

Installing SmartOS the not-smart way, manually!
===

Anyway, if you are like me and want to undertake a fool's errand, I will say that the manual install isn't too bad and was very educational for me since I don't know much about SmartOS, ZFS, the Solaris-based administration commands, or anything like that. Boring over the SmartOS boot scripts was a great way of figuring out where everything lived and why it's important!

I don't think trying to regurgitate my notes is really useful, since I imagine these scripts could change at any given time. But I will provide some pointers!

I also will probably not be able to explain every Solaris bit since I am still learning about it myself!

* The install script is located over at https://github.com/joyent/smartos-live/blob/master/overlay/generic/smartdc/lib/smartos_prompt_config.sh, which you should find in your SmartOS machine at `/smartdc/lib/smartos_prompt_config.sh`
* There was very little you needed pulled in from other scripts at this time. The bulk of the beginning scripts are for printing the fancy prompts and setting shell variables to capture user input. Occasionally they will do something important in exchange to input, but for the most part the code that actually _does_ stuff is at the end.
* You don't need to initialize the network card! You'll not be able to poll an NTP server to grab the current time, but as of this writing the install doesn't really _need_ the ethernet otherwise and you can set the time manually using `date` for now.
* The script is pretty self-documenting, explaining its rationale as it goes along.
* The one thing I had to figure out how to do was have the server report the maount of RAM it had ... I did that using `sysinfo`, which I used to configure the amount of dump space in the `/zones/dump` ZFS volume (I guess in Linux we would call this the core dump partition?)  and the amount of swap space in `/zones/swap`.

I know it's a distraction, but I felt really good about installing Solaris manually because it forced me to use the commandline tools (which are very different from the Linux and OSX ones I'm used to) and I feel less afraid of administering it now :D
