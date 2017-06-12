---
title: How to set up a virtual pfSense router on a homelab virtualization server running FreeBSD and Bhyve
layout: post
date: 2017-06-11
---

Motivation
-
I have been interested in setting up my own "home lab". A "home lab" to me means I will have my own server infrastructure and possibly even non-trivial networking infrastructure in my own home, including separate LANs to keep my publicly facing self-hosted applications separate from my internal wireless network or video game consoles and such.[^1]

I have chosen, for better or for worse, to run my infrastructure on FreeBSD because I feel it is more approachable a UNIX to explore intimately as an enthusiast network infrastructure engineer and relatively junior DevOps person (I recently switched over after a decade being an unremarkable but experienced Software Developer).

Glossary
-
*warning: these are not incredibly accurate*

```
<dl>
	<dt>pfSense</dt>
	<dd>a firewall/router project built off FreeBSD. A linux alternative would be dd-wrt.</dd>
	<dt>FreeBSD</dt>
	<dd>an open source unix clone with a different design philosophy than the more commonly-known Linux.</dd>
	<dt>Firewall</dt>
	<dd>A software or hardware application that filters all network traffic coming it to keep malicious traffic out allow good traffic through.</dd>
	<dt>Network bridge</dt>
	<dd>A device that lets traffic pass through it unadultered. Useful for passing ethernet traffic from one physical media to another.</dd>
	<dt>Network Router and/or Network Gateway</dt>
	<dd>A device that sits as a single point separating an outside network from an "inside"/"private" network. There is a real difference between the two terms but I don't know what it is.</dd>
	<dt>Network Switch</dt>
	<dd>A switch is a device that, uh, basically is like multiple bridges where traffic from any port can be sent to any other port. I don't want to explain how it does this, so let's just say it's magic because the way switches work are actually hella interesting and fascinating. If you're a keener, contrast ethernet switches against ethernet hubs.</dd>
</dl>
```

Approach
-

I have, as most DSL Internet consumers do, a DSL modem and a home router. Unfortunately it does not support VLANs, which will be the building block of how I'll achieve my aforementioned network separation even if I won't talk about it in this post. This means I have to replace it with something better. Since I had already invested in a nice beefy server for running VMs on, it made a kind of sense[^2] to run my router as a Virtual Machine and pick up a nice enterprise-grade switch that could handle VLANs. I obviously need dedicated hardware to plug my networked devices into and out of, but the hardware device known as a router seems to be a combination of that switch hardware plus firewall and routing software, both of which are software and thus virtualizable.

Equipment Required
-

You will need the following hardware to do this:

1. A Cable/DSL modem that isn't a router, or a Cable/DSL modem that can be set to "bridge mode"[^3]
2. A virtualization server to run the router OS on. As you can imagine, put as much RAM as you can into it. pfSense needs at least 512M of RAM allocated to its VM and a gigabytes of disk space.
3. Two network cards. This can either be a single quad or dual-port network card, or two network cards. One of these will be fed from the DSL modem, the other will feed into your switch.
4. A network switch, because presumably you want to be running many devices on your network :)

A diagram of how this will look like from a 3000 ft view is as follows:

**Big Bad Internet** => **Cable/DSL Modem** => **Ethernet Port 0 on Virtual Machine ** => ** pfSense Virtual Machine ** => **Ethernet Port 1 on Virtual Machine** => **Physical Switch**

Security Implications
-

One of the nice things about having a dedicated hardware firewall is that there is a real physical isolation. With a firewall running in a virtual machine there are the following security concerns if not more:

* The outside world internet has physical access to the hardware firewall and nothing more. With a VM router, I am giving the outside world physical access to my underlying server.[^4]
* If a bad person breaks my firewall appliance, they have access to my firewall or router and that is bad. If a bad person breaks my firewall virtual machine, they theoretically have access to my entire virtualized infrastructure, not just over ethernet but with administrative control over the virtualization software as well.
* Even if I configure my network settings properly, I could accidentally misconfigure them to let the outside world network cross over and be given access to my internal network since it's all separated by software. This is much harder to misconfigure on a physical device.

Assumptions
-

* You have a running, working FreeBSD install
* You have two ethernet devices named **bge0** and **bge1**. These will be different depending on what drivers your network devices use.
* You are using **bge1** as your connection to the big bad Internet. It will purely work as a bridge from the cable modem to the router VM, if anything else enabled access to it it would be exposing your internal network to the hostile world and be a misconfiguration and generally very bad news :)
* You are using **bge0** as your internal network. It will double as both the server's access to the local network (and wider internet) and as the physical link between your router VM and your physica internet switch.


[^1]: An alternative strategy would be to use servers like Digital Ocean and/or Amazon instead of running a bunch of dodgy services from my house. For reasons unexplained I decided to go the self-hosted route :)

[^2]: The next-next section will point out why the security-minded are having a heart-attack at this statement, don't worry, wait for it :)

[^3]: Bridge mode means that the model will act purely as a physical connection between the coax/phoneline signal your telco feeds you and the ethernet connection your computer needs. This is different from the "gateway mode" you normally run routers in, where it does the routing and firewalling for you.

[^4]: I can sort of get around this via a sneaky virtualization technique called "PCI Device Bypass". Sadly I could not get it to work, but I'll write up instructions for it sometime anyway.
