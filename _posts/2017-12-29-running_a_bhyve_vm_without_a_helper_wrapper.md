---
title: Running a pfSense instance in FreeBSD/bhyve without any wrappers like iocage or vm
layout: post
date: 2017-12-31 00:03 EDT

---

I am an obstinate git, which is why I'm currently running my internet router/firewall as a [pfSense](https://pfsense.org) VM guest on a FreeBSD server.

As you can imagine, this makes setting up my home network very ... interesting, because first I have to set up FreeBSD and then my router?

[Bhyve](https://bhyve.org) is the FreeBSD-native hypervisor for running virtual machines. I know that FreeBSD supports being a host for something more industrial like [Xen](https://www.xenproject.org) but that is pretty damn heavyweight and actually loses a bunch of functionality in FreeBSD compared to running it on Linux.

Bhyve is a pretty low-level tool. Just as with FreeBSD jails, one doesn't normally interact with the built-into-FreeBSD low-level commands directly, one uses a helper package to streamline the VM creation/operation/configuration process, and to start up VMs on bootup, etc...

But those tools aren't installed on a freshly-installed FreeBSD install, and I didn't want to first have to configure my networking stack to work using a consumer hardware router device and then reconfigure everything back into my FreeBSD-server-centric system.....

OK what I mean to say is that I had this working and then I screwed up my server and I needed to rebuild it and wanted to figure out a lazier way to do so :)

Anyway I'll just spit out the raw instructions and then maybe I'll explain what's going on. I don't think any of this isn't explained in the FreeBSD handbook or by reading example bhyve scripts, but I wanted a nice simple handy reference for the future.

#### Things you'll need

* A freshly installed FreeBSD server.
* A copy of the pfSense installation media. _In particular_, you'll need a version set to communicate via serial port, not a VGA/video card, because Bhyve doesn't support emulating a VGA card yet without a whole lotta hassle that I don't know how to do if it's even doable :) Currently this means when you're at the [download](https://www.pfsense.org/download/) page, choose the USB Memstick installer and the "Serial" console option when downloading the image.
* It's probably best you grab an offline copy of the FreeBSD handbook, because you're going to want to keep [this page](https://www.freebsd.org/doc/handbook/virtualization-host-bhyve.html) handy. You can grab PDF and epub and HTML copies of the handbook in a variety of languages over at https://download.freebsd.org/ftp/doc/
* Have your network devices set up as I recommended over on this [blog post](http://robotdisco.github.io/2017/07/03/how_to_set_up_a_virtual_pfsense_router_on_a_homelab_virtualization_server_running_freebsd_and_bhyve.html). If you're wondering why I have two network devices set up, the blog post will explain why. If you're setting up pfSense on FreeBSD and don't grok the bigger picture you should probably read that post anyway :)

I guess in theory you need the Internet to get these resources, as well as a USB stick, but it can't be turtles all the way down!

#### Setting up pfSense.

1. Install FreeBSD, that's out of scope for this blogpost.
2. Copy the pfSense install file somewhere, to `/home/<yourname>/pfsense.img` or whatever.
3. Use the handbook page mentioned above to set up bhyve's necessary kernel modules, create a disk image for pfSense, and set up a `bridge` interface via `ifconfig`.
4. For the record, I'm going to assume you created the disk image file over at `/vm/pfsense.dsk`. But it doesn't matter, and feel free to use ZFS as per the handbook (I did!)
5. Also for the record, I recommend a Hard Drive image size of at least two gigabytes although make it as large as you want, disk space is cheap (also it's not mentioned at that point in the handbook but ZFS does have the option of sparse volumes where the image only takes up as much space as was ever most needed by the VM, but I feel that's a bit out of scope to get into.)
6. Run the following: `bhyveload -c /dev/nmdm0A -m 1G -d /home/<yourname>/pfsense.img`
7. Run the following: `bhyve -m 1G -AHP -s 0,hostbridge -s 1,lpc -s 2:0,virtio-blk,/vm/pfsense.dsk -s 3:0,virtio-net,tap0 -s 4:0,virtio-net,tap1 -s 31:0,virtio-blk,/home/<yourname>/pfsense.img -l com1,/dev/nmdm0A pfsense`
8. Now switch to a different console in FreeBSD via Alt-F2 or whatever.
9. Run the following: `cu /dev/nmdm0B` Voila! You now have access to the initial pfSense console setup! Finish setting things up until it restarts, which will be a lie, restarting actually terminates the bhyve process.
10. I don't know if you actually need this for real, but kill the still-existing VM resource state via `bhyvectl --destroy --vm=pfsense`. Don't worry! Because you've installed things to disk, all settings and such still remain on /vm/pfsense.img! (It's weird I don't know why that resource state still exists. I guess it's like the machine is suspended at the point it's telling its (virtualized) hardware to shut down?
11. Run `bhyveload -c /dev/nmdm0A -m 1G -d /vm/pfsense.dsk`. Note that we're now giving the HDD image!
12. Run `bhyve -m 1G -AHP -s 0,hostbridge -s 1,lpc -s 2:0,virtio-blk,/vm/pfsense.dsk -s 3:0,virtio-net,tap0 -s 4:0,virtio-net,tap1 -l com1,/dev/nmdm0A pfsense`. Note that we aren't referencing the install image now!
13. As before, switch to a different console via Alt-F2. I think the FreeBSD terminal is dumb enough that it has been waiting around for input forever and will catch the newly rebooted VM's serial output automatically, but run step 9) again if that isn't the case.
14. Maybe you can configure the LAN and WAN ports entirely via the console! I had to use the web configurator over at https://<your router's LAN IP> which meant a laptop and a properly set up Ethernet switch was necessary to set up my DSL credentials :(

But you now have a working pfSense .... until you restart FreeBSD :) At this point you have enough internet to configure your FreeBSD host to have its own IP address (again see my [previous pfSense/FreeBSD post]()) and install your Bhyve wrapper of choice (at this time of writing, `iohyve` or `vm-bhyve` have been used by me and I like both equally). Usually what I do is use their command-line tools to set up a new permanent pfsense instance, kill the one I was manually running, and move the /vm/pfsense.img file where the wrapper exists the disk drive to be. You'll have to read up on your preferred tool to do that, but at least you have the internet now :)

#### OK so what the hell is bhyveload and what are all these params?

I'm not going to explain the stuff that feels obvious from the manpage, but .... ok, for some reason, bhyve uses `bhyveload` to load the FreeBSD kernel into the VM and prep it for booting. Did I mention that pfSense is based on FreeBSD? I guess now you know :)
Afterwards you run `bhyve` which actually creates the generic virtual hardware and such. My understanding is that if you wanted to boot into Linux for example, instead of `bhyveload` you use something called `bhyve-grub` which is the equivalent for loading the Linux kernel but I don't know how it works or if shares the same arguments.

Anyway the `bhyveload` kernel loader you're basically telling it the amount of memory you want the system to have, the FreeBSD console device to use for terminal output, and boot media. You then tell all that to the `bhyve` program as well.

The `bhyve` manpage explains -AHP pretty well and why you want to use them. All those `-s` params basically set up hardware. In something like `-s 3:0,virtio-blk,/vm/pfsense.dsk`, it's basically saying "Hardware device 3 is a virtualized block (i.e. disk) device, and here's an argument (which here means the actual disk image file used to store the emulated hard drive's data)." I don't think it matters which device is in which "slot", as it were, I can use basically any number up to at least 31 I think.
The `hostbridge` and `lpc` devices are core and necessary to any VM. The rest are obviously network and disk images.
The `-l` flag seems to set up the console device.

Anyway hopefully that explains a little bit more about how Bhyve works. It feels pretty simple now, but the documentation is always written in that nerd style that's way more imposing than it needs to be!

#### PCI passthru caveat

If you use PCI passthru on any of your networking devices, as I did a la [this page](http://robotdisco.github.io/2017/07/03/pci_passthru_in_bhyve.html), you'll have to add `-S` to both the `bhyveload` and `bhyve` commands.
