---
layout: post
title: The Great Homelab Migration - Part 5 - Ringer
date: 2017-10-02 12:00:00 -0600
author: Derek Winters
categories: homelab
---

Things are starting to get moving on migrating the lab away from my legacy Windows environment. I've got HomeSeer moved to a NUC, proxmox installed on the old HomeSeer hardware, and a shiny new bind server up and running. The next big requirement is DHCP. This one was easy to name. Coming again from Fulton Brewery, it's time to build ringer.

## Installation

Like letitroll, I don't expect to need much from this VM. It's getting a 32GB (proxmox default) root drive, 1 CPU, and dynamic RAM from 512-1024MB. Installation took about 5 minutes, updates took another 5. I could get a template up to speed this process up, but it's just making me want foreman and puppet more, so I'm keeping things manual for now. I also really can't wait to get puppet managing SSH keys so I don't need to keep typing passwords when hopping between machines. During the install I used my new DNS server instead of the legacy one, just to confirm it's working, and to have one less system to change later. Once the install was finished, I added the new server to my DNS server because it doesn't make any sense to have so much fun with server names and not use them.

This is another new install for me, so I'm going to see how this guide from [itzgeek.com](http://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/install-and-configure-dhcp-server-on-centos-7-ubuntu-14-04.html) works for me.

## Packages

Just one package to install this time.

`yum install -y dhcp`

## Config Files

### /etc/dhcp/dhcpd.conf

Grab the default config and put it where it belongs. I'm a fan of default configs built into packages like this, one less step to forget to have a backup of the default config.

`cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf`

Let's take a look at the default again before we move on.

```
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#

# option definitions common to all supported networks...
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

# Use this to enble / disable dynamic dns updates globally.
#ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

# No service will be given on this subnet, but declaring it helps the 
# DHCP server to understand the network topology.

subnet 10.152.187.0 netmask 255.255.255.0 {
}

# This is a very basic subnet declaration.

subnet 10.254.239.0 netmask 255.255.255.224 {
  range 10.254.239.10 10.254.239.20;
  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
}

# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.

subnet 10.254.239.32 netmask 255.255.255.224 {
  range dynamic-bootp 10.254.239.40 10.254.239.60;
  option broadcast-address 10.254.239.31;
  option routers rtr-239-32-1.example.org;
}

# A slightly different configuration for an internal subnet.
subnet 10.5.5.0 netmask 255.255.255.224 {
  range 10.5.5.26 10.5.5.30;
  option domain-name-servers ns1.internal.example.org;
  option domain-name "internal.example.org";
  option routers 10.5.5.1;
  option broadcast-address 10.5.5.31;
  default-lease-time 600;
  max-lease-time 7200;
}

# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.

host passacaglia {
  hardware ethernet 0:0:c0:5d:bd:95;
  filename "vmunix.passacaglia";
  server-name "toccata.fugue.com";
}

# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.
host fantasia {
  hardware ethernet 08:00:07:26:c0:a5;
  fixed-address fantasia.fugue.com;
}

# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.

class "foo" {
  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
}

shared-network 224-29 {
  subnet 10.17.224.0 netmask 255.255.255.0 {
    option routers rtr-224.example.org;
  }
  subnet 10.0.29.0 netmask 255.255.255.0 {
    option routers rtr-29.example.org;
  }
  pool {
    allow members of "foo";
    range 10.17.224.10 10.17.224.250;
  }
  pool {
    deny members of "foo";
    range 10.0.29.10 10.0.29.230;
  }
}
```

Looks like the default is going to have a lot to remove. After following the guidelines in the config, here is the new config. Not that it really matters, but I removed the MACs and IPs.

```
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#

# option definitions common to all supported networks...
option domain-name "home.derekwinters.com";
option domain-name-servers 172.16.0.10;

default-lease-time 600;
max-lease-time 7200;

# Use this to enble / disable dynamic dns updates globally.
#ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

subnet 172.16.0.0 netmask 255.255.255.0 {
  range 172.16.0.220 172.16.0.249;
  option routers 172.16.0.1;
}

host freenas {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}

host sawtooth {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}

host polestar {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}

host ruination {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}

host homefront {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}

host derekphone {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}

host amyphone {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}

host firetvwifi {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}

host printer {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}

host rainmachine {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}

host expat {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.0;
}
```

### Services

Before starting the new DHCP server I've got to make sure to disable my old one so there aren't any conflicts. Since I'm still using the old DNS server I don't expect issues with systems dropping from the domain, but I can test and monitor that later if it happens. To be safe I deactivated the scope on the Windows server, stopped the service, and disabled the service.

```
[root@ringer ~]# systemctl enable dhcpd
[root@ringer ~]# systemctl start dhcpd
[root@ringer ~]# systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2017-08-03 20:33:04 CDT; 5s ago
     Docs: man:dhcpd(8)
           man:dhcpd.conf(5)
 Main PID: 17169 (dhcpd)
   Status: "Dispatching packets..."
   CGroup: /system.slice/dhcpd.service
           └─17169 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Aug 03 20:33:04 ringer.home.derekwinters.com dhcpd[17169]: Copyright 2004-2013 Internet Systems Consortium.
Aug 03 20:33:04 ringer.home.derekwinters.com dhcpd[17169]: All rights reserved.
Aug 03 20:33:04 ringer.home.derekwinters.com dhcpd[17169]: For info, please visit https://www.isc.org/software/dhcp/
Aug 03 20:33:04 ringer.home.derekwinters.com dhcpd[17169]: Wrote 0 deleted host decls to leases file.
Aug 03 20:33:04 ringer.home.derekwinters.com dhcpd[17169]: Wrote 0 new dynamic host decls to leases file.
Aug 03 20:33:04 ringer.home.derekwinters.com dhcpd[17169]: Wrote 0 leases to leases file.
Aug 03 20:33:04 ringer.home.derekwinters.com dhcpd[17169]: Listening on LPF/eth0/d6:38:c3:ca:93:ed/172.16.0.0/24
Aug 03 20:33:04 ringer.home.derekwinters.com dhcpd[17169]: Sending on   LPF/eth0/d6:38:c3:ca:93:ed/172.16.0.0/24
Aug 03 20:33:04 ringer.home.derekwinters.com dhcpd[17169]: Sending on   Socket/fallback/fallback-net
Aug 03 20:33:04 ringer.home.derekwinters.com systemd[1]: Started DHCPv4 Server Daemon.
```

### Firewall

```
[root@ringer ~]# firewall-cmd --add-service dhcp
success
[root@ringer ~]# firewall-cmd --add-service dhcp --permanent
success
```

### Test

Time to test it out. I didn't set up a reservation for my desktop, and after releasing the reservation and requesting a new address, I'm not in the new 172.16.0.220-249 pool. I can also see I'm getting an address from the new server.

```
[derek@chainsaw ~]$ cat /var/lib/dhclient/dhclient.leases

lease {
  interface "enp2s0";
  fixed-address 172.16.0.220;
  option subnet-mask 255.255.255.0;
  option routers 172.16.0.1;
  option dhcp-lease-time 600;
  option dhcp-message-type 5;
  option domain-name-servers 172.16.0.10;
  option dhcp-server-identifier 172.16.0.254;
  option domain-name "home.derekwinters.com";
  renew 5 2017/08/04 01:42:10;
  rebind 5 2017/08/04 01:42:10;
  expire 5 2017/08/04 01:42:10;
}
```

Time to test a reservation as well. I'd also like to be able to keep track of what systems still need my old DNS server to make it easier to know when I can retire it. I added a reservation for my desktop, and put a DNS option in for just that reservation.

```
host chainsaw {
  hardware ethernet 00:00:00:00:00:00;
  fixed-address 172.16.0.204;
  option domain-name-servers 172.16.0.11;
}
```

Restart dhcpd on the server.

```
[root@ringer ~]# systemctl restart dhcpd
```

Release, renew.

```
[derek@chainsaw ~]$ sudo dhclient -r
[derek@chainsaw ~]$ sudo dhclient
```

And things look good (extra info removed).

```
[derek@chainsaw ~]$ ip addr show

2: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.204/24 brd 172.16.0.255 scope global dynamic enp2s0
       valid_lft 571sec preferred_lft 571sec
```

```
[derek@chainsaw ~]$ cat /etc/resolv.conf 
# Generated by NetworkManager
search home.derekwinters.com
nameserver 172.16.0.11
```

```
[derek@chainsaw ~]$ cat /var/lib/dhclient/dhclient.leases 

lease {
  interface "enp2s0";
  fixed-address 172.16.0.204;
  option subnet-mask 255.255.255.0;
  option routers 172.16.0.1;
  option dhcp-lease-time 600;
  option dhcp-message-type 5;
  option domain-name-servers 172.16.0.11;
  option dhcp-server-identifier 172.16.0.254;
  option domain-name "home.derekwinters.com";
  renew 5 2017/08/04 02:48:53;
  rebind 5 2017/08/04 02:53:25;
  expire 5 2017/08/04 02:54:40;
}
```

## Conclusion

Now that I know everything is working as I expect, I'll go back and make a few extra changes. The main change I'm going to make is setting the default DNS value to the new server, and override it only on the hosts that still need it for active directory. For everything that isn't joined to the domain, I should be able to use the new Linux servers for DNS and DHCP. My next project will be getting dynamic DNS working between the two servers so that I only have one system to update for all the new servers I'll be creating as I continue to migrate to my new lab. So far, I've got plenty of room left on insurrection for the migration.
