---
layout: post
title: The Great Homelab Migration - Part 6 - Dynamic DNS
date: 2017-11-06 12:00:00 -0600
author: Derek Winters
categories: homelab
---

Now that I have a DNS server and a DHCP server, it's time to connect them together. I'm going to give [this guide](https://www.dnsknowledge.com/tutorials/centos-tutorials/bind-9/howto-setup-dynamic-dns-ddns/) a shot.

## Setup

### DHCP

Add the line `'allow client-updates;'` to `/etc/dhcp/dhcpd.conf`, copy the key file over from the DNS server and include it in the config, and add the zones that will be updated.

```
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#

# option definitions common to all supported networks...
option domain-name "home.derekwinters.com";
option domain-name-servers 172.16.0.11;

default-lease-time 600;
max-lease-time 7200;

# Use this to enble / disable dynamic dns updates globally.
ddns-update-style interim;
ddns-domainname "home.derekwinters.com";
update-static-leases on;
ddns-rev-domainname "in-addr.arpa";
ddns-updates on;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

#ignore client-updates;
allow unknown-clients;

include "/etc/rndc.key";

zone 0.16.172.in-addr.arpa {
  primary 172.16.0.11;
  key rndc-key;
}

zone home.derekwinters.com {
  primary 172.16.0.11;
  key rndc-key;
}

subnet 172.16.0.0 netmask 255.255.255.0 {
  range 172.16.0.220 172.16.0.249;
  option routers 172.16.0.1;
}

host freenas {
  hardware ethernet 40:16:7e:64:bc:19;
  fixed-address 172.16.0.20;
  option domain-name-servers 172.16.0.10;
}

host sawtooth {
  hardware ethernet e8:ab:fa:50:20:00;
  fixed-address 172.16.0.40;
}

host polestar {
  hardware ethernet 00:15:5d:00:64:02;
  fixed-address 172.16.0.60;
}

host ruination {
  hardware ethernet c4:e9:84:70:ff:56;
  fixed-address 172.16.0.104;
  option domain-name-servers 172.16.0.10;
}

host homefront {
  hardware ethernet 00:15:5d:00:0a:06;
  fixed-address 172.16.0.109;
  option domain-name-servers 172.16.0.10;
}

host derekphone {
  hardware ethernet 8c:f5:a3:85:ff:87;
  fixed-address 172.16.0.107;
}

host amyphone {
  hardware ethernet 8c:f5:a3:4f:0a:f5;
  fixed-address 172.16.0.108;
}

host firetvwifi {
  hardware ethernet 10:ae:60:02:58:63;
  fixed-address 172.16.0.110;
}

host printer {
  hardware ethernet 40:b0:34:8c:47:17;
  fixed-address 172.16.0.50;

}

host rainmachine {
  hardware ethernet a8:80:38:20:df:4c;
  fixed-address 172.16.0.51;
}

host expat {
  hardware ethernet f4:4d:30:6f:6e:ca;
  fixed-address 172.16.0.70;
}

host chainsaw {
  hardware ethernet 48:5b:39:c3:dd:c9;
  fixed-address 172.16.0.204;
}
```

### DNS

Include the key and allow it to update the zones.

```
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// See the BIND Administrator's Reference Manual (ARM) for details about the
// configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {
#	listen-on port 53 { 127.0.0.1; };
#	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	allow-query     { localhost;172.16.0.0/24; };

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-enable yes;
	dnssec-validation yes;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.iscdlv.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

include "/etc/rndc.key";

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

zone "home.derekwinters.com" IN {
	type master;
	file "fwd.home.derekwinters.com.db";
	allow-update { key rndc-key; };
};

zone "0.16.172.in-addr.arpa" IN {
	type master;
	file "0.16.172.db";
	allow-update { key rndc-key; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

### SELinux

Now we need to tell SELinux to allow the named user to manage the zone files.

```
[root@letitroll ~]# chcon -R -t dnssec_trigger_var_run_t '/var/named/'
```

### NTP

Make sure time is in sync on both hosts.

```
yum install -y ntp
ntpdate 0.centos.pool.ntp.org
systemctl enable ntpd
systemctl start ntpd
```

### Services

Restart named and dhcpd.

## Test

There are a few things that I didn't make reservations and DNS entries for. One of those is my access point. I wasn't able to finish testing the first night working on this, but when I came back a few days later I checked the zone files on letitroll.

In /var/named/fwd.home.derekwinters.com.db, I found a new entry.

```
[root@letitroll ~]# cat /var/named/fwd.home.derekwinters.com.db

$ORIGIN home.derekwinters.com.
$TTL 300	; 5 minutes
EX6150			A	172.16.0.222
```

And another in /var/named/0.16.172.db.

```
$TTL 300	; 5 minutes
222			PTR	EX6150.home.derekwinters.com.
```

So it looks like DNS entries are working. The TTL is very low though, which must be a default value. I added a setting value for ddns-ttl and will monitor named for changes.

```
ddns-ttl 86400;
```

## Conclusion

One less config file to manage. I love that everything in Linux is in a config file somewhere, but that doesn't mean I like to spend all day maintaining them. I ran into a few issues while getting this up and running, but the above config files are the end result. A few quick notes for anyone that could run into the same issues as me.

- Including /etc/rndc.key works like you may expect if you've worked with programming languages that allow includes. The rndc.key file has a key {} definition, and the name of that definition was rndc-key. Every guide I found would include the file, then reference key rndckey, not dndc-key. It was easy to skim that when I looked at rndc.key the first time.
- Originally, DHCP was trying to update the reverse zone for 104.0.16.172.0.16.172.in-addr.arpa. Changing the ddns-rev-domainname to "in-addr.arpa" fixed that issue.
- Various guides I found had various levels of values required in /etc/dhcp/dhcpd.conf. What I found above was what got my setup working. Some guides included 'ignore client-updates;' while subsequent forum posts recommended removing it to get the updates working.

I'm not sure what project I'll be tackling next. I have a list of around 30 services that I want to set up, and I'm getting close to needing to pull the trigger on retiring the domain for good. We'll see if that comes next or not.
