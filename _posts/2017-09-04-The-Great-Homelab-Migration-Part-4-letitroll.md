---
layout: post
title: The Great Homelab Migration - Part 4 - letitroll
date: 2017-09-04 12:00:00 -0600
author: Derek Winters
categories: homelab
---

Time to get a new DNS server up and running. I didn't have as much luck with finding a really good name for the primary DNS servers I'll be adding to the network, but I do have two really similar names that should work well for the eventually dual-DNS setup I plan to have. The first DNS server I have to add in will be called letitroll, named after the beer from Indeed.

## Installation

The proxmox install for the letitroll VM was perfectly smooth, and after installing CentOS 7.3 and getting it up to date on patches, it's time to get bind set up and running. It's been a very long time since I touched bind, so I'll be using a guide from Linux Pit Stop to keep me on track.

### Packages

I'll need three packages to get things up and running.

`sudo yum install -y bind bind-libs bind-utils`

### Config Files

#### /etc/named.conf

First up, I always like to keep the default config as a reference, and it's always a good idea to backup a config before making changes.

`sudo cp /etc/named.conf /etc/named.conf.backup`

Now let's dive in and take a look at the default config, and start making changes.

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
        listen-on port 53 { 127.0.0.1; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { localhost; };

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

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

From the file we can see by default bind will only be listening on localhost, and allow queries from localhost. This should be changed to allow the service to listen on any interface, and allow queries from my local network.

```
#       listen-on port 53 { 127.0.0.1; };
#       listen-on-v6 port 53 { ::1; };


        allow-query     { localhost;172.16.0.0/24; };
```

Next up, time to add our forward and reverse zones.

```
zone "home.derekwinters.com" IN {
        type master;
        file "fwd.home.derekwinters.com.db";
        allow-update { none; };
};

zone "0.16.172.in-addr.arpa" IN {
        type master;
        file "0.16.172.db";
        allow-update { none; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

#### /var/named/fwd.home.derekwinters.com.db

```
$TTL 86400

@ IN SOA primary.home.derekwinters.com. root.home.derekwinters.com. (
	1708022200 ;Serial
	3600 ;Refresh
	1800 ;Retry
	604800 ;Expire
	43200 ;Minimum TTL
)
;Name Server Information
@ IN NS primary.home.derekwinters.com.
;IP address of Name Server
primary IN A 172.16.0.11
;A - Record HostName To Ip Address
chainsaw     IN A 172.16.0.204
expat        IN A 172.16.0.70
freenas      IN A 172.16.0.20
homefront    IN A 172.16.0.109
insurrection IN A 172.16.0.30
inversion    IN A 172.16.0.1
letitroll    IN A 172.16.0.11
polestar     IN A 172.16.0.60
printer      IN A 172.16.0.50
rainmachine  IN A 172.16.0.51
ruination    IN A 172.16.0.104
sawtooth     IN A 172.16.0.40
titan        IN A 172.16.0.10
traitor      IN A 172.16.0.63
;CNAME record
warrior   IN CNAME freenas.home.derekwinters.com.
```

#### /var/named/0.16.172.db

```
$TTL 86400
@ IN SOA primary.home.derekwinters.com. root.home.derekwinters.com. (
	1708022200 ;Serial
	3600 ;Refresh
	1800 ;Retry
	604800 ;Expire
	86400 ;Minimum TTL
)
;Name Server Information
@ IN NS primary.home.derekwinters.com.
;Reverse lookup for name server
11 IN PTR primary.home.derekwinters.com.
;PTR Record IP address to HostName
1 IN PTR inversion.home.derekwinters.com.
10  IN PTR titan.home.derekwinters.com.
11  IN PTR letitroll.home.derekwinters.com.
20  IN PTR freenas.home.derekwinters.com.
30  IN PTR insurrection.home.derekwinters.com.
40  IN PTR sawtooth.home.derekwinters.com.
50  IN PTR printer.home.derekwinters.com.
51  IN PTR rainmachine.home.derekwinters.com.
60  IN PTR polestar.home.derekwinters.com.
63  IN PTR traitor.home.derekwinters.com.
70  IN PTR expat.home.derekwinters.com.
104 IN PTR ruination.home.derekwinters.com.
109 IN PTR homefront.home.derekwinters.com.
204 IN PTR chainsaw.home.derekwinters.com.
```

#### Firewall

```
[root@letitroll ~]# firewall-cmd --add-service dns
success
[root@letitroll ~]# firewall-cmd --add-service dns --permanent
success
```

#### Service

```
[root@letitroll ~]# systemctl start named
[root@letitroll ~]# systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-08-02 21:38:57 CDT; 2s ago
  Process: 19229 ExecStart=/usr/sbin/named -u named $OPTIONS (code=exited, status=0/SUCCESS)
  Process: 19226 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf -z /etc/named.conf; else echo "Checking of zone files is disabled"; fi (code=exited, status=0/SUCCESS)
 Main PID: 19231 (named)
   CGroup: /system.slice/named.service
           └─19231 /usr/sbin/named -u named

Aug 02 21:38:57 letitroll.home.derekwinters.com named[19231]: zone 0.in-addr.arpa/IN: loaded serial 0
Aug 02 21:38:57 letitroll.home.derekwinters.com named[19231]: zone 1.0.0.127.in-addr.arpa/IN: loaded serial 0
Aug 02 21:38:57 letitroll.home.derekwinters.com named[19231]: zone 0.16.172.in-addr.arpa/IN: loaded serial 1708022132
Aug 02 21:38:57 letitroll.home.derekwinters.com named[19231]: zone 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa/IN: loaded serial 0
Aug 02 21:38:57 letitroll.home.derekwinters.com named[19231]: zone localhost.localdomain/IN: loaded serial 0
Aug 02 21:38:57 letitroll.home.derekwinters.com named[19231]: zone home.derekwinters.com/IN: loaded serial 1708022131
Aug 02 21:38:57 letitroll.home.derekwinters.com systemd[1]: Started Berkeley Internet Name Domain (DNS).
Aug 02 21:38:57 letitroll.home.derekwinters.com named[19231]: zone localhost/IN: loaded serial 0
Aug 02 21:38:57 letitroll.home.derekwinters.com named[19231]: all zones loaded
Aug 02 21:38:57 letitroll.home.derekwinters.com named[19231]: running
[root@letitroll ~]# systemctl enable named
Created symlink from /etc/systemd/system/multi-user.target.wants/named.service to /usr/lib/systemd/system/named.service.
```

#### Notes

Make sure there is a . at the end of all the domain names in the config files. I also found that the Serial for the config files can't be as long as I originally wanted. I was hoping to have the full year, month, day, as well as the hour, minute, and an additional integer because I tend to make a lot of typos, so minute wasn't enough for me. For now, I'm going to use two-digit year, month, day, and hour, minute, although I might change the minute to a increasing integer instead. However you decide to make your serial number doesn't matter as long as it continues to increase. I just like date (and from what I remember from years ago it's a common method for obvious reasons) so that is what I use.

### Testing

Now that the service is up and running, it's time to make sure it's working. On my desktop I changed my DNS (currently from DHCP) to the new server, and ran a lookup on a system I know is in the new server.

```
[derek@chainsaw ~]$ cat /etc/resolv.conf 
# Generated by NetworkManager
search home.derekwinters.com
nameserver 172.16.0.11
```

```
[derek@chainsaw ~]$ dig expat

; <<>> DiG 9.10.5-P2-RedHat-9.10.5-2.P2.fc25 <<>> expat
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 15068
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;expat.				IN	A

;; AUTHORITY SECTION:
.			10800	IN	SOA	a.root-servers.net. nstld.verisign-grs.com. 2017080202 1800 900 604800 86400

;; Query time: 123 msec
;; SERVER: 172.16.0.11#53(172.16.0.11)
;; WHEN: Wed Aug 02 21:45:28 CDT 2017
;; MSG SIZE  rcvd: 109
```

```
[derek@chainsaw ~]$ dig expat.home.derekwinters.com

; <<>> DiG 9.10.5-P2-RedHat-9.10.5-2.P2.fc25 <<>> expat.home.derekwinters.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32054
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;expat.home.derekwinters.com.	IN	A

;; ANSWER SECTION:
expat.home.derekwinters.com. 86400 IN	A	172.16.0.70

;; AUTHORITY SECTION:
home.derekwinters.com.	86400	IN	NS	primary.home.derekwinters.com.

;; ADDITIONAL SECTION:
primary.home.derekwinters.com. 86400 IN	A	172.16.0.11

;; Query time: 0 msec
;; SERVER: 172.16.0.11#53(172.16.0.11)
;; WHEN: Wed Aug 02 21:45:31 CDT 2017
;; MSG SIZE  rcvd: 110
```

```
[derek@chainsaw ~]$ dig google.com

; <<>> DiG 9.10.5-P2-RedHat-9.10.5-2.P2.fc25 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59305
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 5

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		300	IN	A	216.58.216.78

;; AUTHORITY SECTION:
google.com.		172401	IN	NS	ns1.google.com.
google.com.		172401	IN	NS	ns2.google.com.
google.com.		172401	IN	NS	ns4.google.com.
google.com.		172401	IN	NS	ns3.google.com.

;; ADDITIONAL SECTION:
ns2.google.com.		172401	IN	A	216.239.34.10
ns1.google.com.		172401	IN	A	216.239.32.10
ns3.google.com.		172401	IN	A	216.239.36.10
ns4.google.com.		172401	IN	A	216.239.38.10

;; Query time: 35 msec
;; SERVER: 172.16.0.11#53(172.16.0.11)
;; WHEN: Wed Aug 02 21:47:33 CDT 2017
;; MSG SIZE  rcvd: 191
```

Looks good! Since I have a few systems still joined to my existing domain, I'm not going to change DNS everywhere so they don't drop off the domain and lose authentication. I also tested to make sure the recursion setting was working properly. Right now my DNS server is actually using titan for DNS still, so my last step will be removing the dependency on titan for my DNS.

```
[root@letitroll ~]# cat /etc/resolv.conf 
# Generated by NetworkManager
search home.derekwinters.com
nameserver 8.8.8.8
nameserver 8.8.4.4
```

## Conclusion

I'm one step closer to retiring the old domain controller and moving over to a full Linux homelab. Next up I think I'll get DHCP up and running on another Linux VM, and hopefully get secure DNS updates working so I can keep from needing to update DHCP and DNS config files any time I make a new server. Hopefully once I get Foreman running, I can also avoid having to edit DHCP as well, and have Foreman managing everything.
