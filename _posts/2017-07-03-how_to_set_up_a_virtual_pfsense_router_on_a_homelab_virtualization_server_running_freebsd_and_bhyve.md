---
title: How to set up a virtual pfSense router on a homelab virtualization server running FreeBSD and Bhyve
layout: post
date: 2017-07-03 01:33 EDT

---

Motivation
-
I have been interested in setting up my own "home lab". A "home lab" to me means I will have my own server infrastructure and possibly even non-trivial networking infrastructure in my own home, including separate LANs to keep my publicly facing self-hosted applications separate from my internal wireless network or video game consoles and such.[^1]

I have chosen, for better or for worse, to run my infrastructure on FreeBSD because I feel it is more approachable a UNIX to explore intimately as an enthusiast network infrastructure engineer and relatively junior DevOps person (I recently switched over after a decade being an unremarkable but experienced Software Developer).

Glossary
-
*warning: these are not incredibly accurate*

**pfSense:** a firewall/router project built off FreeBSD. A linux alternative would be dd-wrt.
**FreeBSD:** an open source unix clone with a different design philosophy than the more commonly-known Linux.</dd>
**Bhyve/Iohyve:** Virtualization software native to FreeBSD. This is similar to KVM on Linux.</ddr>
**Firewall:** A software or hardware application that filters all network traffic coming it to keep malicious traffic out allow good traffic through.</dd>
**Network bridge:** A device that lets traffic pass through it unadultered. Useful for passing ethernet traffic from one physical media to another.</dd>
**Network Router and/or Network Gateway:** A device that sits as a single point separating an outside network from an "inside"/"private" network. There is a real difference between the two terms but I don't know what it is.</dd>
**Network Switch:** A switch is a device that, uh, basically is like multiple bridges where traffic from any port can be sent to any other port. I don't want to explain how it does this, so let's just say it's magic because the way switches work are actually hella interesting and fascinating. If you're a keener, contrast ethernet switches against ethernet hubs.</dd>

Approach
-

I have, as most DSL Internet consumers do, a DSL modem and a home router. Unfortunately it does not support VLANs, which will be the building block of how I'll achieve my aforementioned network separation even if I won't talk about it in this post. This means I have to replace it with something better. Since I had already invested in a nice beefy server for running VMs on, it made a kind of sense[^2] to run my router as a Virtual Machine and pick up a nice enterprise-grade switch that could handle VLANs. I obviously need dedicated hardware to plug my networked devices into and out of, but the hardware device known as a router seems to be a combination of that switch hardware plus firewall and routing software, both of which are software and thus virtualizable.

Equipment Required
-

You will need the following hardware to do this:

1. A Cable/DSL modem that isn't a router, or a Cable/DSL modem that can be set to "bridge mode"[^3]
2. A virtualization server to run the router OS on. As you can imagine, put as much RAM as you can into it. pfSense needs at least 512M of RAM allocated to its VM and a gigabyte of disk space.
3. Two network cards. This can either be a single quad or dual-port network card, or two network cards. One of these will be fed from the DSL modem, the other will feed into your switch. 
4. A network switch, because presumably you want to be running many devices on your network :)

A diagram of how this will look like from a 3000 ft view is as follows:

**Big Bad Internet** => **Cable/DSL Modem** => **Ethernet Port 0 on Virtual Machine** => **pfSense Virtual Machine** => **Ethernet Port 1 on Virtual Machine** => **Physical Switch**

Security Implications
-

One of the nice things about having a dedicated hardware firewall is that there is a real physical isolation. With a firewall running in a virtual machine there are the following security concerns if not more:

* The outside world internet has physical access to the hardware firewall and nothing more. With a VM router, I am giving the outside world physical access to my underlying server.[^4]
* If a bad person breaks my firewall appliance, they have access to my firewall or router and that is bad. If a bad person breaks my firewall virtual machine, they theoretically have access to my entire virtualized infrastructure, not just over ethernet but with administrative control over the virtualization software as well.
* Even if I configure my network settings properly, I could accidentally misconfigure them to let the outside world network cross over and be given access to my internal network since it's all separated by software. This is much harder to misconfigure on a physical device.

Assumptions
-

* You have a running, working FreeBSD install
* You are using ZFS -- a bunch of tooling assumes it, sorry :'(
* You have two ethernet devices named **bge0** and **bge1**. These will be different depending on what drivers your network devices use.
* You are using **bge1** as your connection to the big bad Internet. It will purely work as a bridge from the cable modem to the router VM, if anything else enabled access to it it would be exposing your internal network to the hostile world and be a misconfiguration and generally very bad news :)
* You are using **bge0** as your internal network. It will double as both the server's access to the local network (and wider internet) and as the physical link between your router VM and your physical internet switch. I like using the 0th interface for the internal rather than external connection because software tends to assume it by default which means you won't accidentally expose yourself over 
* You know how to set up your switch. I will handwave some instructions but apparently VLAN terminology is very different depending on the type of switch you use.

Anyway, let's get to it.

Instructions
-

Tune system settings
--

Add the following lines to your `/etc/sysctl.conf` file, which I guess configures kernel runtimeoptions.

* `net.link.bridge.pfil_onlyip=0` (This allows non-IP traffic to flow over a bridge interface. I am unsure if this is needed.)
* `net.link.tap.up_on_open=1` (This enables the VM network to be enabled implicitly on demand)

Add the following lines to your `/etc/loader.conf`, which I guess configures kernel boot-time options.

* `vmm_load="YES"` (I guess this is some kind of virtualized memory thing)
* `nmdmm_load="YES"` (BHyve uses null modems to communicate over virtualized serial ports to the VMs)
* `if_bridge_load` (I guess this enables bridge interfaces?)
* `if_tap_load` (I guess this enables tap interfaces)

Add the following to your `/etc/rc.conf` file, which configures system services at startup.
* `if_bge0="dhcp"` (This enables my first network card but doesn't configure it on the host OS.)
* `if_bge1="up"` (This enables my second network card. It is important for security reasons for the outside-facing network card to be a passive passthru and not allow IP-based access to the host system)
* `cloned_interfaces="bridge0 bridge1 tap0 tap1"` (This is where we define all our virtual interfaces.)
* `defaultrouter=192.168.1.1` (This is the default address of the pfSense virtual machine)
* `ifconfig_bridge0="inet 192.168.1.2 netmask 255.255.255.0 addm tap0 addm bge0"` (bridge virtual network 0 and my first network port)[^5]
* `ifconfig_bridge1="addm tap1 addm bge1"` (bridge virtual network 1 and my second network port)

At this point you'll want to reboot.

Install necessary software, fetch, decompress, install pfSense ISO
--

* `pkg install iohyve` (This is the higher-level VM manager software, the core low-level binary `bhyve` is a little involved)
* `iohyve setup pool=<your ZFS partition>` (this primes the bhyve layout)
* `curl -O https://nyifiles.pfsense.org/mirror/downloads/pfSense-CE-2.3.4-RELEASE-amd64.iso.gz` (this was the latest file at time of writing)[^6]
* `gunzip pfSense-CE-2.3.4-RELEASE-amd64.iso.gz`
* `sudo iohyve cpiso ./pfSense-CE-2.3.4-RELEASE-amd64.iso` (this installs the ISO into iohyve's library of installation software`

Create pfSense vm:
--

* `sudo iohyve create firewall0 4G` (give VM about 4G of disk space)
* `sudo iohyve set firewall0 boot=1 ram=1G tap=tap0,tap1` (most VMs only need one fake network, but we need two here. Also, boot this image on startup. Also, give the VM 1G of RAM)
* `sudo iohyve getall firewall0` (it's nice to familiarize yourself with the settings of your VM)
* `sudo iohyve isolist` (this is how you see the ISOs you've downloaded for installing software)
* `sudo iohyve install firewall0 pfSense-CE-2.3.4-RELEASE-amd64.iso` (this boots up the VM from the CD, as it were)
* `sudo iohyve console firewall0` (this lets you access the machine, where you will configure everything.

The pfSense install GUI
---

I found that I had to pick some specific options in order to make sure everything worked.

1. Quick/Easy Install (I am not a brave person)
2. OK (I think it complained to me about not having enough RAM or whatever. Or it wanted to confirm the we would wipe the entire hard disk or something.)
3. Embedded Kernel (no VGA console, keyboard). <-- CHOOSE THIS, IT IS VERY IMPORTANT. BHYVE CURRENTLY DOES NOT ALLOW VGA ACCESS, YOU HAVE TO USE THE SERIAL PORT TO CONFIGURE STUFF
4. Reboot (this shuts off the machine rather than reboots it, for whatever reason. You'll have to `iohyve start firewall0` to bring it back up)

Post-install fun
--

The default IP tends to be 192.168.1.1. The default user/pass combination is user: "admin" and password: "pfsense", all lowercase.

If you do `iohyve list` you should see output as follows:
```
Guest  VMM? Running rcboot? Description
firewall0 YES NO YES <some date>
```

This indicates that the firewall VM has been provisioned (VMM), isn't running, but is configured to start when the host machine books.

Initial configuration
--

This is where we configure your pfSense install with real values.

1. `iohyve start firewall0`
2. `iohyve console firewall0`
3. (Don't set up VLANs unless you know what you're doing)
4. Configure vtnet1 to be your WAN interface (can't configure actual DSL login information yet :/ If you're using cable/DHCP for your WAN you're perhaps golden :D)
5. Configure vtnet0 to be your LAN interface
6. Your firewall image will now reboot or start up or something
7. Unplug any ethernet cables, especially your external one. It helps your misconfigured interfaces time out easier.
8. When you see the menu pop up, plug everything back in. Now is a good time to turn on the SSH interface.
9. Send your browser to 192.168.1.1
10. Configure your new firewall as you would configure a hardware one (including DSL login information!)

Mission Accomplished
-

At this point, you're ready to set up your switch and VLANs, which I feel is out of scope and perhaps I'll write about them later.




[^1]: An alternative strategy would be to use servers like Digital Ocean and/or Amazon instead of running a bunch of dodgy services from my house. For reasons unexplained I decided to go the self-hosted route :)

[^2]: The next-next section will point out why the security-minded are having a heart-attack at this statement, don't worry, wait for it :)

[^3]: Bridge mode means that the model will act purely as a physical connection between the coax/phoneline signal your telco feeds you and the ethernet connection your computer needs. This is different from the "gateway mode" you normally run routers in, where it does the routing and firewalling for you.

[^4]: I can sort of get around this via a sneaky virtualization technique called "PCI Device Bypass". Sadly I could not get it to work, but I'll write up instructions for it sometime anyway.

[^5]: My machine has a _third_ ethernet port that I used my old router with while setting this up, and I eventually set up VLANS bypassing this. So I'm kind of guessing here how you'd end up allowing LAN access while letting most traffic pass through.

[^6]: It occurs to me that maybe you should download this image before mucking with your internet setup, :/
