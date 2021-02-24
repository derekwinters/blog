---
layout: post
title: The Great Homelab Migration - Part 1 - The Plan
date: 2018-01-01 12:00:00 -0600
author: Derek Winters
categories: homelab
---

The day has come. It's time to get rid of the Active Directory dependencies on the FreeNAS server. I'll be removing all the AD permissions on FreeNAS, removing the AD connections, and re-sharing all the existing shares. After that, I'll reconfigure Kodi to use the new shares and get all the clients reconfigured.

## Warrior

### Users

First things first, I want some unique users local to the FreeNAS server to connect with. I may hook in to a new identity management (FreeIPA, etc) system later, but I want to kill of Active Directory first. I created readonly and readwrite user accounts for now. Kodi will use readonly, but Homefront will use readwrite.

### Permissions

On my dataset (/mnt/DataSet1), I need to change the local file permissions to use the new accounts. On the Storage tab under Volumes, select the dataset and at the bottom of the window a set of options appears. "Change Permisisons" and set owner user and group to readwrite. I kept the mode the same (775), change permission type to Unix, and "Set permission recursively." This takes a few minutes to go through the 8.7TB of data on /mnt/DataSet1.

### Shares

Now on the Sharing tab under Windows (SMB), it's time to delete all the existing shares and create some new ones. I created two shares for the /mnt/DataSet1/Public path, Public-RO and Public-RW. Two shares wasn't necessary and I may change that later, but the RO also has the "Export Read Only" checkbox checked as an added safeguard.

I did want to use NFS here, but after a lot of troubleshooting and reading, NFS and Kodi don't seem to play very well. I still plan to use NFS for all my other systems, but for now it looks like Kodi will need SMB shares. Since my Kodi shares are direct SMB paths (to ensure the libraries on all the clients see the exact same files in the same paths to prevent duplicates), I couldn't use the local mount workaround a lot of forum posts suggested.

In the services tab, I also needed to uncheck to "domain logons" checkbox.

### Active Directory and Kerberos

On the Directory tab under Active Directory, I disabled the AD connection with the "Enabled" checkbox near the bottom. I also removed the home.derekwinters.com Kerberos Realm on that page.

## Titan

Now that all the old shares are gone, I need to stop trying to use them. I had a group policy mapping \\warrior\Public to the P: drive on all my systems, so I disabled that. I took this time to disable all the other GPO settings I had (SNMP, RDP, etc). Since I'm very close to being done with Active Directory, I wanted to get that part over with now.

## Kodi

I did all the initial testing and configuration on my desktop. I changed the sources.xml file to use the new shares, and moved what used to be in passwords.xml into the path information in the sources.xml file.

```
<sources>
    <programs>
        <default pathversion="1"></default>
    </programs>
    <video>
        <default pathversion="1"></default>
        <source>
            <name>Movies</name>
            <path pathversion="1">smb://172.16.0.20/Public-RO/Media/Movies/Movies-720p/</path>
            <path pathversion="1">smb://172.16.0.20/Public-RO/Media/Movies/Movies-1080p/</path>
            <path pathversion="1">smb://172.16.0.20/Public-RO/Media/Movies/Movies-3d/</path>
            <allowsharing>true</allowsharing>
        </source>
        <source>
            <name>Television</name>
            <path pathversion="1">smb://172.16.0.20/Public-RO/Media/Television/Episodes/</path>
            <path pathversion="1">smb://172.16.0.20/Public-RO/Media/Television/Seasons/</path>
            <allowsharing>true</allowsharing>
        </source>
    </video>
    <music>
        <default pathversion="1"></default>
    </music>
    <pictures>
        <default pathversion="1"></default>
    </pictures>
    <files>
        <default pathversion="1"></default>
        <source>
            <name>Fusion</name>
            <path pathversion="1">http://fusion.xbmchub.com/</path>
            <allowsharing>true</allowsharing>
        </source>
    </files>
</sources>
```

I relaunched Kodi and let the library update, then ran the clean library process to remove all the old files that are part of the old paths. I considered looking into how the Kodi database was set up and changing the database rows instead, so that I could keep all my watched history. I ended up being a little impatient, and since it's summer I don't have a lot of shows I'm watching to keep track of, so I just created a new library and marked all the old stuff watched.

A quick note on my library structure. I ripped all my movies into folders based on the quality (used to do 720p, now with more space I have 1080p but never went back and ripped the old stuff), just so the directories don't become unmanageable. Right now they aren't big enough to need to categorize any further, but I've seem people use both genre and alphabetical sub-directories to help with that. For television, anything I record OTA or download goes into episodes. If it's a show I think I'll rewatch, I move it into seasons when I have a full season of episodes. Otherwise when I'm done with a show I go into the Episodes folder and delete the episodes. I really hate having half a season when I go to rewatch something, and the episodes/seasons directories help keep me on top of what I need.

## FireTV

```
./adb kill-server
./adb start-server
./adb connect 172.16.0.110
./adb push passwords.xml /sdcard/android/data/org.xbmc.kodi/files/.kodi/userdata/passwords.xml
./adb push sources.xml /sdcard/android/data/org.xbmc.kodi/files/.kodi/userdata/sources.xml
```

## Homefront

On homefront I simply deleted the old P: mapping and created a new one with the path \\warrior\Public-RO and the user warrior\readonly.

## Ruination

Same thing on ruination, except I don't need the share so I didn't remap it. I'll just use the Kodi files to get the media directories.

All the config files for Kodi are in %AppData%, and I simply replaced them with the ones I used on the other two servers. The database connections don't change, so I only need to change sources and passwords.

## Conclusion

Now that all the Kodi instances are off the old shares and permissions, I'm almost done with Active Directory. I have three systems still on the domain right now, smokejumper (desktop), homefront (torrents), and ruination (HTPC). I plan to move the torrent VM to Linux, so when that gets built I can shut down that one. I plan on wiping out the smokejumper partition completely, so that isn't a concern.

That leaves ruination. I'm not sure what my plan is yet. There are a few things we use that for that I need to make sure would work with Linux if I moved it. I have a Flirc IR-to-USB adapter that lets my Logitech Harmony remote directly control Kodi, and I love this. We also use this computer to view the nursery camera a lot, and the Foscam plugin I believe only works in Internet Explorer. I know I can get to it from VLC too, but I need to find a way to make that simple to keep my wife happy, and having her remember long cgi-bin commands won't fly. I think I would either make a shortcut or save it in VLC, or what I'd love to do is get either BlueIris, or build a simple web page for viewing the streams around our house. At one point I also got it working within Kodi, but that broke a long time ago and I haven't looked into why. Long story short, ruination will probably be joining a workgroup until I can get time to find solutions to a few issues.

That leaves me with basically one final project left before I can retire Titan. Next chance I get, I'll be building a new VM for a torrent client so I can keep contributing my storage capacity to all the Linux ISOs out there. When that is complete, I'll shut down traitor (temporary LibreNMS host), move ruination to a workgroup, and shut down my 5 year old Active Directory domain.
