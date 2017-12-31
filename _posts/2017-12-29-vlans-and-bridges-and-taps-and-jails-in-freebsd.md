---
title: VLANs and Bridges and Taps and Jails in FreeBSD
layout: post
date: 2017-12-29 22:01 EDT

---

This is sort of an embarrassing post; I've struggled for months (one could say, I had a _devil of a time_) figuring out something that ought to be very trivial in FreeBSD; that is, how do I have VMs running in bhyve and jails talking to each other and reachable from the outside world based on VLANs?

The good news is I ended up scouring the internet to find lots of interesting FreeBSD networking trivia that was never quite the answer. I even invested a weekend reproducing my home-lab setup in Linux (via Proxmox) to figure out whether and how that "it just works" GUI would configure the networking setup I thought I needed.

### Here are my findings. No; first, here was my desire. No; first, here's the ultimate point.

1. **Don't create `vlan` network interfaces off `tap` devices. Use real network cards to which you feed all the VLAN traffic you require.**

### OK. Now second, here is what I _wanted to do_, for background.

1. I have a FreeBSD server that I intend to be the private cloud running the various Virtual machines and jails that comprise my aspiration home-lab and private cloud.
2. For reasons that are both outside the scope of this post and are an incredibly indicator of hubris, I run my firewall as a [pfSense](https://www.pfsense.org/) virtual machine. Its virtual network interface `tap0` exposes a trunk (a connection carrying all VLAN traffic in my system, tagged) connection to my Cisco switch via a physical network interface, called `bge0`.
3. I have an additional network card, `em0` that I have assigned my server's address in. It has fancy remote-control functionality built into the hardware (video/keyboard/mouse over Ethernet) that I feel shouldn't be accessible from outside my administration VLAN, so I intended it to be accessed from an untagged Cisco port (it doesn't tag any traffic; this has a nice-side effect that every time I destroy my networking setup I can plug my laptop into another untagged port and rely on basic networking to fix things :)
4. I have a few VLANs for home and internet service use; let's call them `vlan20` and `vlan50` since those are the VLAN IDs I have chosen. I have a variety of jails attached to them via a software bridge (`bridge20` and `bridge50`, for consistency's sake.) In case it isn't clear. For those not in the know, these are basically filters on top of a trunk connection that handles multiple VLAN connections that creates effectively an untagged interface. My guess is that this is a more secure option than allowing a VM or jail to explicitly tap into the fire-hose directly, since an attacker could create a filter onto my administration VLAN and that would be bad, no?
5. I have a docker server for things that I can't figure out (or couldn't be bothered to) mangle into a FreeBSD jail. Usually this is prepackaged services that I want to use that assume everybody runs docker and Linux since they are the victorious commodities now. In theory these VMs connect to the respective bridges and everything just works.
6. Nothing worked. I could either hook the VMs into the same bridge as the jails and _those_ services could talk, but then the VM wouldn't get any internet access. Alternatively I could attach the VMs to the main trunk interface `tap0` and explicitly attach to the relevant VLANS within the guest OSes (not ideal, but if it works...) which would allow the instances to access the outside world, but not the jail servers. I could only have one or the other, which sucked, since the reverse proxy that governs access and terminates SSL for these docker containers is a jail :/

That was the quandary I fought over for many months instead of doing something useful. Again, for hubris reasons, I didn't want to "just" run Docker/Linux, my ego is too fragile for that :) (also I find Linux incredibly disorienting as a newbie server administrator and FreeBSD is more amenable to play and exploration and discovering concepts personally.)

### Here is a bunch of interesting things about FreeBSD networking. These may or may not be true, but this is how I've wrapped my brain around networking concepts:

1. `ifconfig` is very unhelpful if you try to do things like bridge unsupported network interfaces, or attach a VLAN pseudo-interface to an unsupported device. The response can often be vague like "Device busy" when what really means is one of the following:
2. You can't create `vlan*` devices off `bridge*` devices, even though that's basically what you do in Linux. You have to configure them to tap off a member device like the physical or software Ethernet devices.
3. FreeBSD forums and man pages keep telling you that if you want to have an assigned IP on a network segment created by bridging various interfaces to something like `bridge0`, then you should assign the IP address to `bridge0` not the member interfaces like `bge0` or `em0` or what have you. In Linux this makes sense because everything just _works_ (you can attach virtual and real interfaces to bridges just fine, you can tell an interface to only care about a particular VLAN that is being bridged, etc...) In FreeBSD this ends up being a source of confusion because bridges don't support a whole bunch of configuration and you have to perform the configuration on a member.
4. If I have a VM with an interface called `tap0` and it's attached via physical port `bge0` to a Cisco switch, in order to provide a VLAN interface to my jails and VMs I have to derive the `vlan` interface off `tap0` not `bge0`. I guess the bridge is smart enough to not bother passing traffic to the switch if it knows the destination is a virtual device as well. Also this isn't a solution because it turns out to be a lie anyway so this doesn't really matter! For reasons I get to later.
5. So the way VLAN tagging works is that a network connection carrying multiple VLANs worth of traffic tags each packet with an additional 4 byte(?) tag indicating what VLAN the network is for. Normally we assume internet packets are 1500 bytes (this is user configurable but that's not relevant since all devices need to match this value which is called MTU which I won't bother explaining.) I don't know if is invisibly handled at a lower hardware level (apparently the layer underneath, internet frames, are larger, 1522 bytes or something, because they contain the internet protocol payload + their own data)
6. When you create a `vlan0` device off a real interface like `bge0` or `em0` (the `bge` and `em` indicate the network card chip, Broadcom and Intel in this case BTW) the resulting untagged VLAN device `vlan0` still has a MTU aka packet size of 1500. Is this normally interesting? I don't know. BUT!
7. When you create a `vlan0` device off a virtual interface like `tap0` the resulting pseudo-interface's MTU is 1496. This implies maybe that we're shaving off the tag and the sacrifice we make is 4 bytes a packet that would otherwise be used for real internet traffic? That seems reasonable, one would think ... but again, this is a lie for reasons I will get to :)
8. If you add interfaces to a bridge device, all of the MTU values of the various interfaces have to be the same. This makes sense in my head. This means that one of these `vlan0` devices created off a `tap0` device won't be bridgeable to a network device with an MTU of 1500 since `vlan0` will have an MTU of 1496. OK, in the case where all of my other bridge members are jails or VMs, I guess it's only fair to change their MTUs with the command `ifconfig bridge0 mtu 1496` and `ifconfig tap1 mtu 1496` and such? Again, this is a lie :)
9. The lie revealed! It turns out that creating a `vlan0` device off a `tap0` device just winds up in some kind of broken zombie network connection that FreeBSD seems to be okay with but no actual traffic passes through! I ran various network diagnostic tools and would see traffic go out the `vlan0` connection but never make it to the trunk connection or to the router or to the jail or what have you! And yet, when I created the `vlan0` off a real device like `em0` or `bge0`, everything worked fine! That was what solved my problem.

**So in short, don't create `vlan0` off `tap` devices :)** I would like to think this is a bug but I have no idea how to even go reporting this. I kind of feel annoyed that my management Ethernet port now receives two other VLAN segments which I worry means that some jerk could exploit the NIC's remote-control features if they exploited enough of my internet services stack, but I could also just buy another network card and deal with it that way.

Lastly, here's a few capsule summaries of:

**The FreeBSD config I wanted**
1. A `bridge0` containing `tap0`, my router VM's networking pseudo-device and `bge0`, my physical uplink to my Cisco switch.
2. An `em0` device that exclusively serves out administrative traffic off VLAN 10 (and doesn't realize it's on a VLAN, the Cisco switch sneakily handles that.)
2. A `vlan20` and `vlan50`, tapping off `tap0`, creating untagged segments for my home and internet services (web-apps, etc...)
3. A bridge `bridge20` containing `vlan20` and whatever jail and VM interfaces needed to be on VLAN segment 20.
4. A bridge `bridge50` containing `vlan50` and whatever jail and VM interfaces needed to be on VLAN segment 50.

**The FreeBSD config I wound up with**
1. A `bridge0` containing `tap0`, my router VM's networking pseudo-device and `bge0`, my physical uplink to my Cisco switch.
2. An `em0` device that receives VLAN 10 untagged (also tagged, but again, assuming VLAN 10 as the untagged default is nice when I screw up my config as I usually do) but also receives VLAN 20 and VLAN 50 tagged.
2. A `vlan20` and `vlan50`, tapping off `em0`, creating untagged segments for my home and internet services (web-apps, etc...)
3. A bridge `bridge20` containing `vlan20` and whatever jail and VM interfaces needed to be on VLAN segment 20.
4. A bridge `bridge50` containing `vlan50` and whatever jail and VM interfaces needed to be on VLAN segment 50.

#### Half-assed glossary:

**Ethernet**: If you have devices (you do) which talk via the Internet (you do) or to each other (you do) they're using this protocol to do so, unless you know better.

**VLAN**: Most consumer devices create _one_ network for all devices connected to them. This is fine for consumer devices, but what if you're a weirdo like me who wants to segment his home media devices (Playstation, Laptop) from the servers it using to serve his webpage and mail to the Internet? The Gaelan could use multiple computers, multiple routers, all independant ... or it could use slightly professional hardware which can send effectively multiple networks over the same physical networking channel by "tagging" each network with a different ID as part of the low-level protocol metadata. Assuming the hardware and low-level software isn't defective, the traffic is virtually segmented which is nice from an isolation and maintenance point of view.

**Network Bridge**: Basically what your router does, Allow multiple devices to talk to each other because they're all wired together. Except if you know anything about networking, it doesn't do any of the fancy stuff like firewalling (denying bad traffic into to from or across your system) or routing (dispatching data from your home network to a network somewhere else on the Internet by knowing who the appropriate entrypoint into the other system is). Except it does do some really interesting low-level stuff that's not really relevant. Also in this case it's all via software dealing with software interfaces not physical hardware. When you hear of a networking hub or switch they're basically bridges except you learn some other interesting implementation details with really important ramifications that affect how your devices are bridged in the abstract.

**Tap device**:  I use this here to effectively mean a virtualized network device my virtual machines are using. Inside the guest system they think it's a network card ... the host OS knows that's bullshit. In some technical way a Tap means something more involved and specific but damned if I know what.

**Jail**: If you know Docker, it's a container. Basically instead of emulating a full machine, the OS basically develops a split contained personality that thinks it is an independant instance. It's arguably not as secure as faking an entire machine, but it's also way more performant and way more light-weight. Jails are just FreeBSD's implementation of containers; Solaris has "Zones", for example, with its own set of implementation-specific details.
