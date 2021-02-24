---
layout: post
title: When Boredom Sets In
date: 2019-04-29 12:00:00 -0600
author: Derek Winters
categories: homelab
---



It’s been over a year since my last post, and honestly not a lot has changed in the lab since then. There were two reasons that I started my last big migration project. The first was a new job. I was transitioning from a Windows admin role back to what made me fall in love with computers, Linux. I was excited to get back into Linux and wanted to reacquaint myself. The second was my wife getting pregnant with our second child, and developing a deep love with sleeping all of the time. Buying a new computer and getting to do the rebuild once our first was asleep kept me busy through those long, boring months.

Then the new baby came, and time disappeared. A year flies by and the homelab goes into KTLO mode for a while. A lot changes in a year though, and the lab does not have the fun things that I’d like to start working on and digging into more, so it’s time for another rebuild. No, baby number three is not on the way this time.

Right now, I don’t have many services running, and definitely nothing shiny or attention grabbing. I have a DHCP server and a BIND server, connected together for dynamic DNS updates. I have a MySQL server running, hosting a database for Kodi (version 18 is really nice). I have a broken LibreNMS server that never really got set up, and I have a seedbox for supporting all the Linux distros out there. I’ve also got an Intel NUC for HomeSeer, and a big FreeNAS server with 21 TB usable sitting on the ZFS equivalent of RAID 10 (pools of mirrors). I’ve also gotten rid of a custom built HTPC and replaced it with a Nvidia Shield, and added another Shield to a second TV. The Amazon Fire TV is still running but showing it’s age (still). I also made a small upgrade in WiFi, and replaced a little Netgear extender with a Ubiquiti AC-Pro.

That’s boring these days. Especially when I like using my homelab to try new things for work, and make cool things. The only convenience I have is the shared Kodi database on three different TVs, connected to a big NAS. The rest is just stuff that needs to be maintained, and that’s what people pay me to do. I don’t get paid to keep the WiFi at home working.

## What the new lab will be

So first, I need to decide what I want my homelab to be for, and what I don’t want it to be for. I don’t want to maintain DNS and DHCP in config files. It was a good learning experience, but besides the few things I had to troubleshoot, and just the general maintenance, I wouldn’t call myself an expert in the subject after setting up a single domain with 20 hosts, and I’ll never get beyond that in a homelab.

I really want to get deeper into containers, and maybe even get a Kubernetes lab running. I have set up Graylog at work, along with ElasticSearch and MongoDB to support it. I’m maintaining RedHat Satellite, and doing a lot with Ansible as of late. We have a lot of Kafka too, and I’d like to learn a little more about that. But of everything we’re doing, Kubernetes is definitely the one I’m most interested in.

## My (initial) plan

Like all of my lab plans, things change a lot as I go through it, and since I actually use all of the services for our day-to-day at home, I have to keep a semblance of stability to keep the cameras connected, the thermostat doing all the little adjustments we like, and the lights magically turning on when you walk into a room.

With that in mind, the first thing I need to do is simplify the tedious administration that I’m doing now, with the added benefit of freeing up some resources. I’m thinking that I will be investing in at least one raspberry Pi and migrate DHCP and DNS over to PiHole. The project looks really cool and gets mentioned online all the time. It’s also an excuse to buy a new toy. That also moves Proxmox from hosting critical services to hosting the nice-to-have seedbox, and frees up a few GB of memory.

At that point I’ll come to another decision about my hypervisor. I chose Proxmox because I had heard a lot about it, and it was relatively simple to set up with my condensed timeline to be done with the lab before the baby arrived. It’s a little boring though, and I wouldn’t mind diving back into the original plan of KVM. Then I circle back to doing a lot of admin tasks keeping that running. I’m also really interested in turning it into a MiniShift host and really digging into that so I can truly compare it to vanilla Kubernetes (which I’m also interested in building from scratch, maybe ona cluster of Pis).

As far as services go, I’d like to add some new ones into the lab. I’m a huge fan of Graylog (we use it to host something like 25TB of logs with almost no administration work). I have more automation ideas for my house too, but I’m starting to run into some walls where a small middleware API service would solve a LOT of my complicated issues. I’d like to get LibreNMS back online and get it actually monitoring things again. We use Ansible Tower at work, and I’d like to stand up AWX and get some of my home config into Ansible, and even look into Graylog or LibreNMS triggering jobs in AWX. I’m interesting in hosting an internal GitLab server that pushes code to my public GitHub. I’m planning on adding more WiFi access points, and more cameras, so more WiFi monitoring and a network video recorder might be in my future.

There are countless other ideas I have, and only time will tell which ones I decide to focus on. First thing first, I need to get something better/easier for DHCP and DNS so I can kill off those servers. Time to start the parade of Amazon deliveries.
