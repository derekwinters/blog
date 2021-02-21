---
layout: post
title: The Great Homelab Migration - Part 1 - The Plan
date: 2017-07-15 12:00:00 -0600
author: Derek Winters
categories: homelab
---

My homelab has grown and changed a lot over the years, but rarely has it gone through as big of a migration as I'm planning now. In college it was a playground for whatever random project I was working on, or crazy idea I came up with. When I graduated and got a job, it became a learning lab for things I found exciting at work or cool things I read or heard about.

At my previous job I was a Windows admin, and as such, I currently have a modest Windows domain with a variety of applications and services that are mostly used for day-to-day operations at home. I've recently switched jobs, however, and have found my way back into the world of Linux administration. Having made my rounds through Windows, and being very excited to be back on the Linux side of the world, it's time to reevaluate my homelab.

## The History

In 2009 I built my first computer, with a dual-core AMD Athlon, 2x2GB RAM, and a 1TB hard drive. I ran Windows for a while, until I got an internship worked closely with a Linux admin. After a brief period of dual-booting, I took the plunge and wiped the drive for Fedora, and picked up a few extra storage drives, some more RAM, and started learning MDADM. I ran this for 1-2 years until 2010 when I couldn't get Starcraft 2 installed, and reluctantly went back to Windows.

In 2011, I got my tax return and like a wise college student, promptly spent it all on a new desktop. Quad-Core AMD Phenom, 4GB RAM, and shuffled some of my hard drives around to get 160GB boot drives on both computers, with now 3x1.5TB for the RAID array. Newegg shopping history is really helping my memory right now. Another 1.5TB drive arrived a few months later for me to successfully learn MDADM expansion with, bringing the storage total to 4.5TB. A capacity I never thought I would fill (spoiler alert, there is never enough) Eventually the Linux server turned into a 2012 Domain Controller, and the 1.5TB were replaced with 2TB drives, and again replaced with 3x4TB WD Reds in a storage pool.

In 2015, the next big addition arrived, FreeNAS. The data was backed up to the old 2TB drives, and another 4TB arrived to create two mirrors for a total usable array of 7.5TB. Sitting around 50% full, I again incorrectly thought to myself "it will be years until I need more."

2016 brought us the purchase of our house and the arrival of our son, and as such, it was time for a new computer and a dive into home automation and security. A budget build from Micro Center with a Pentium G3258 took on the installation of HomeSeer, and I was off to the races installing a WiFi camera and randomly replacing light switches and door locks all over the house. The FreeNas server was becoming angry at constantly battling 80-90% full, so on the new family/house budget, I repurposed the old 2TB drives to give it some wiggle room, bringing the usable total to 10.5TB.

The early months of 2017 were again a time of an angry FreeNAS server, but a Best Buy deal on two 8TB Easystores increased the storage by around 5TB. While looking over the S.M.A.R.T. stats on the two remaining 2TB drives, however, panic set in as a slew of prefail/old age warnings filled the screen, along with run times crossing the 4-5 year mark. Two new 4TB drives arrived shortly after that to complete the rebuild. The pools in FreeNAS are now 2x4TB, 2x4TB, 2x4TB, and 2x8TB for a total usable capacity of 18TB sitting at 47% used. This time I know I won't need to increase that anymore. No one needs more than 18TB of storage at home.

## Today

That brings us to today. I have a domain controller and hypervisor running on nearly 9-year-old hardware, with a phyiscal limit at two-cores and 8GB RAM. The entire environment is Windows (except for FreeNAS and a small VM running LibreNMS). I recently shrunk the 500GB SSD in my desktop and installed Fedora 25 (a week before 26 was released so now I have an upgrade coming) to go back to a dual boot while I waited to arrive at a plan to rebuild it all. I'm trying to stay rational here and not just buy an R730 with 512GB RAM and dual 8-core procs and go crazy, but I have a lot of projects that I would like to test at home, and I have almost no capacity left unless my desktop becomes another always-on hypervisor.

### Existing Hardware

It's nothing fancy, but it has gotten the job done so far.

I moved away from my whiskey naming convention in college and now use beers. Originally I used mostly Left Hand Brewing, but I'm starting to branch out to others looking for names I like. I'm using mostly Minnesota breweries for any new stuff I build.

#### Warrior

- FreeNAS 9.10u5
- FreeNAS 11 just released with VM support, but I'm hesitant after Corral, and I'd at least need to add some RAM to make that a possibility. It's not off the table.

#### Chainbreaker

- Windows 10
- HomeSeer
- Pentium G3258
- 8 GB RAM
- This all appears to support virtualization, but could probably use a RAM increase.

#### Titan

- Windows Server 2012 R2
- Domain Controller
- Athlon dual-core
- 8GB RAM
- OLD. It could be useful to keep around for a while, but all of my "production" workload needs to get away from this before it dies.

##### Titan VMs
- Homefront
  - Always-on VM for supporting the Linux ISO community
  - Windows
- Traitor
  - LibreNMS
  - CentOS
- Polestar
  - MySQL database for Kodi
  - CentOS

##### Smokejumper/Chainsaw
- Smokejumper - Windows 10
- Chainsaw - Fedora 25
- Desktop

##### Ruination
- Home Theater PC
- Not very powerful, but handles streaming from online and FreeNAS, albeit with some excessive fan noise on occasion.

##### Amazon Fire TV gen1
- Bedroom streaming TV
- Starting to show it's age

##### Inversion
- Cisco RV120W
- Alright router, don't use nearly all the features it has that I thought were SO important
- WiFi disabled

##### Sawtooth
- Foscam 19821P
- Nursery camera

##### Netgear AC1200 Access Point
- Better than the Cisco's 802.11n, but has a hard time covering the house and streaming 1080P

##### Misc
- HP Envy 4520 printer
- HP rack switch - unused (too loud to sit next to my desktop)
- Netgear rack switch - unused (too loud to sit next to my desktop)
- Broken/Old laptops
- 1990s desktops I can't get myself to throw away

### The End Goal

I've been keeping an ever-growing list of things I'd like to learn/try at home on a Trello board. At this point, I have decided I do not plan to migrate any existing services (except HomeSeer) but instead will be starting with a clean slate, and a 100% (or 90%) Linux home network. I'd also like to get more active in community projects, whether that ends up being publishing more to my GitHub, finding ways to work with Linux projects like Fedora (and possibly learning more development), or just blogging more often. With all that in mind, here is the list of things I'd like to get working at home, or migrate from the Windows version I currently use.

I tend to gravitate towards the common enterprise technologies so that what I learn at home and work are interchangeable. Windows at home worked great for that for a long time, but now that I'm working with Linux and am pretty comfortable with the Windows stuff, it's time to switch. I'll be sticking to the RHEL-like stuff for the same reason as I used Windows domain technologies instead of home products or technologies like workgroups.

I also am always considering upgrading to a business internet package and killing off my hosting plan. Paid hosting little cheaper than the upgraded (but slower) internet, but could be a fun project. With the money I spend on hardware, it's probably a good idea. Hosting isn't getting any cheaper either.

#### Virtualization

##### oVirt

For the RHEL-like reason above, I'm nearly settled on oVirt as the virtualization replacement for Hyper-V.

#### Identity Management

##### FreeIPA

Goodbye Windows domain, hello FreeIPA. This will go well with my beer naming convention as well.

#### Monitoring / Management
- LibreNMS
- Graylog
- Foreman
- Puppet
- DHCP
- DNS
- phpIPAM

#### Networking / Security
- pfSense
- BlueIRIS
- Ubiquiti
- Misc
- Squid Proxy
- Print Server (HP Envy 4520)

### The Plan

I still don't know how I'm going to accomplish this, but it's time to pull the trigger and get started. I'll be updating the plan as I go, but I'll do my best to blog my way through it all.

To start the process off, I just purchased an Intel NUC with a dual-core celeron, 8GB RAM, and 120GB SSD to host HomeSeer and free up chainbreaker for bigger things. This is also going to hopefully be one of the only Windows computers I have on my network. I'll be rebuilding it with the chainbreaker name (because I like that for it's purpose, unless chainbreaker becomes my wireless controller VM), but it will no longer be domain joined. I'm going to have to reconfigure the shares on warrior a little bit since I have some of the logging and backups for HomeSeer going there now using domain credentials.

Once the chainbreaker hardware is free, I'll be getting my feet wet with oVirt. I'll probably start with a FreeIPA VM, then maybe look into Foreman/Puppet so I can speed up the deployment of the rest of my VMs later. With the number of times I changed that last sentence as I thought more about my plans though, I really can't say what my plan is yet beyond moving HomeSeer to the lower-power NUC and stashing it away.

I'm also thinking about knocking down a small chunk of the wall between the laundry room and a basement bedroom to reclaim some of the closet as a mini shelf/rack for all the computers that are currently making way too much noise and heat in my small office. It would also be a great wire drop location for my eventual WiFi upgrade, and has relatively easy access to my panel to add a new circuit. I'm not sure if I like that plan yet, but I know these computers need to find a better home.

### Conclusion

That about wraps up the introduction post. I'm hoping to stick with the blogs as I go, if for no other reason than so I can look back in a few years at how much time and effort I put into doing exactly what I do at work, but spending money on it instead of getting paid for it. I guess that means I picked the right career.
