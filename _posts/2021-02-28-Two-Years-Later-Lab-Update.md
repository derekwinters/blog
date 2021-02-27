---
layout: post
title: Two Years Later - Lab Update
date: 2021-02-28 19:00:00 -0600
author: Derek Winters
categories: homelab
---

In the last post I talked about how I've done a lot of renovation around my house. Part of that was a renovation of my office, which is where my lab has been for a long time. First I just had a few PCs on an old desk. After that I moved them onto a shelving system I put in the closet in the office, to clear up some floor space.

The problem with that was I couldn't close the bifold doors because of the heat, and because of the layout of the room there was never room between my desk and the doors when they were open. This meant I was always dealing with a combination of fan noise and excessive heat in the small room.

I decided it was time for another redo. I put in sliding doors in the closet to fix the access issue, I made some changes to networking in the basement to allow me to move the computers into my laundry/utility room, and I downsized the lab again to retire some more aging PCs that weren't needed anymore. At this point, honestly, I'm not sure how much this would be considered a homelab. It's more of mix of systems to handle some basic networking/services, and not much experimentation anymore. Over the past few years I've found myself having less interest in the homelab and more interest in my home automation system.

## Inventory

### Warrior | Freenas

I still have my Freenas server, although it's a bit out of date. Hard drives are all still running fine but I'll probably be making a purchase soon to get a spare or two for the inevitable drive failure. It's still sitting around 25TB usable, in the Freenas version of RAID 1. It has 4 8TB drives and 4 4TB drives in a pool of mirrors. The 4TB drives are getting pretty old.

### Expat | HomeSeer

This is the same Intel NUC as before, still running fine. I'm long overdue for the upgrade to HomeSeer 4 which came out early 2020.

### pihole | Pi 3

Sometime in late 2019 I retired the DNS and DHCP VMs and replaced them with a pihole. I'm really happy with it. It's easy to use and works well. I moved all my static leases over, which was a really simple process.

### Seedbox | Pi 4

After retiring the DNS and DHCP VMs, the only active VM left was the seedbox. I bought another Pi and set up transmission. I don't have any RSS feeds or any automation/scripting around this, and I'm not sure if I will or not.

### Ubiquiti Dream Machine

I've been considering Ubiquiti for a long time. The big catalyst for this upgrade was moving the servers out of the office closet. The room is in the middle of the basement, so it was a good location for the single AC-PRO that I've had for a few years. When I moved things to the end of the house in the utulity room, it was finally far enough to not reach our upstairs TV at all. Before moving, I would struggle to stream 1080P from Freenas, but everything else worked well enough. When I went to pick up the Pi at Micro Center, I finally made the plunge. I bought a Dream Machine and an AP-AC-Lite. I ran conduit from the laundry into the garage and then another run into the attic, and I mounted the AC-Pro in the upstairs ceiling, and the AC-Lite on the opposite end of the basement. I've got capacity now to stream a 4K movie to any TV in the house, although right now the only 4K TV I have is hardwired right back to the Dream Machine.

### NVidia Sheilds

I love these things. Although they are pretty expensive for a streaming device, I've got them on all three main TVs in the house now: family room (4k), living room (1080p), and bedroom (1080p). I have a 720p TV in my office and the kitchen, and those have Roku Expresses. I can't load Kodi on those, but YouTube TV has been good enough for that.
