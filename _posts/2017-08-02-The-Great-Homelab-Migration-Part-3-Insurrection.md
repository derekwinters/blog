---
layout: post
title: The Great Homelab Migration - Part 3 - Insurrection
date: 2017-08-02 12:00:00 -0600
author: Derek Winters
categories: homelab
---

The NUC migration for my HomeSeer install has been running well for about a week, and it's time to re-purpose the previous hardware for bigger things.

I'm not going to claim that I can see the future, but I can't ignore the fact that two weeks after deciding to start this migration project, the power supply fan and two case fans stopped working in titan today. I was in my office when I heard a big spike in fan noise and looked over to see the top exhaust fan standing still. Server is still on, so I RDP over and shut it down cleanly to determine my next move. A quick run to Micro Center and I'm heading home with a new power supply and case fan. At this point, I only thought the case fan went out, and was just going to replace it. I knew my power supply in insurrection only had two SATA power cables, so instead of using the possibly incendiary adapters they had at Micro Center, I picked up a new 80 PLUS 500W power supply with 6 SATA power cables for $35.

So I get home, open the case up and blow out the dust, and start replacing the fan. I also picked up a new 3.5" to 2.5" SSD mount, and take out my 5.25" to 2.5" SSD mounts in titan. The boot drive gets mounted at the bottom of the case, and the single data drive gets moved to the 3.5" adapter. Now I've got a boot drive at the bottom of insurrection, and two data drives for all the future VMs. I turn on the computer and am disappointed to find another case fan was also not running, and the power supply fan isn't spinning. This is a lot worse than I thought but luckily I have a new PSU. The new one still goes into insurrection, but now the old PSU from chainbreaker can hopefully keep titan running just a little longer.

It wouldn't surprise me if my wife suspects sabotage, and I wouldn't blame her. I tend to enjoy spending money on computers.

## The Name

It may not come as a surprise to those who have worked with me, but I take names and conventions pretty seriously. When dealing with thousands of computers, I think it's really important to cram as much information about a systems purpose in a 15-character host name. When dealing with a fun project at home, it's really important to quietly laugh to myself about how clever I am when I need to type in the name of a computer on my network.

I've bounced a few names around for the new hypervisor in my house. Originally it was going to be stranger (now retired from Left-Hand Brewing) because I planned to create a new dedicated Hyper-V server and virtualize all the services running on it, including the domain controller. No one wants a domain-joined hypervisor that hosts it's own domain controller, hence stranger.

After moving away from Hyper-V, I still kept stranger on the list unless something better came up. I've settling (at least for my first round of testing) on using Proxmox as my hypervisor now, and will be naming the new underdog after the beer Insurrection from Fulton Brewery.

## Proxmox Setup

### Installation

I used the Gnome Multi Writer to write the proxmox 5.0 ISO to the USB stick I used previously. This time, no issues getting it to boot properly (must be a Windows thing...).

My first run at the install failed with an error message: "command 'chroot /target dpkg --force-confold --configure -a' failed with exit code 1 at /usr/bin/proxinstall line 385." The [first result](https://forum.proxmox.com/threads/chroot-target-dpkg-error.3874/)] on Google for the error, although old, indicated an issue with the host name and a special character being the cause. [Another result](https://forum.proxmox.com/threads/so-here-is-the-thing.35931/) says the installation requires the network to be connected. That one sounds a lot more accurate, considering my box is off to the side un-cabled. Let's try this again with a network cable.

The next run at the install finished in under 5 minutes.

After the server boots back up after the install, I'm presented with a prompt to visit https://i.p.ad.dr:8006/, or log on to the shell. Off to the web UI.

Looks like an empty install is using around 700MB of RAM and 1.26GB for the OS.

### Test Virtual Machine

Time to poke around a little and get familiar with the new system. From what I can see, the system, although not optimized for what I want to do, should be fully functional. I'm going to test that by spinning up a new VM.

Looks like we get a few options on the first page. Where the VM is going to be hosted, and ID number, a name, and a resource pool (currently unavailable).

Lots of supported OS types. We'll be testing with a CentOS image.

As expected, it doesn't know about my folder of ISOs I never told it about. I'll skip this step now, and figure out how to attach an ISO to a VM after it's created.

Defaults look good here for a test.

Sure, why not.

Let's see what dynamic memory looks like in proxmox.

I have no plans to use anything except bridged mode for any of my VMs.

A nice summary of the VM about to be created.

A new VM is offline and ready for a test installation.

### Storage Configuration

Since I will be redoing most of my storage and authentication on my whole network, I'm going to set up insurrection with no external dependencies for now. That means getting the two additional SSDs mounted and available for storage of VMs and ISOs during the transition to the new infrastructure.

```
fdisk /dev/sdb
  n
  1
  t
  8e
  w
fdisk /dev/sdc
  n
  1
  t
  8e
  w
vgdisplay
vgextend pve /dev/sdb1
vgextend pve /dev/sdc1
vgdisplay
pvscan
lvdisplay
lvextend /dev/pve/data /dev/sdb1
lvextend /dev/pve/data /dev/sdc1
```

Now I've got 260GB of usable space to start building new VMs to replace the services running on titan.

## Conclusion

So far I haven't even scratched the surface of proxmox, but what I've done was very easy and it gets me the base to continue my lab migration. Next up I'll be getting some new servers up to get the core services moved over. I'll be getting a server up for DNS, DHCP, MySQL, and torrents. After that I'll need to change permissions on my FreeNAS shares to make sure everything stays accessible when I finally retire the Active Directory domain. Once the domain is shut down and titan is retired, I'll likely be looking at adding a second node to proxmox and see how the clustering options compare to what I'm used to with Hyper-V and ESX.
