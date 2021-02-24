---
layout: post
title: The Great Homelab Migration - Part 7 - Highroad
date: 2017-12-04 12:00:00 -0600
author: Derek Winters
categories: homelab
---

Next up on the chopping block is polestar. Polestar is currently running a small mySQL database that is used by Kodi to keep my library in sync across multiple clients. I can stop watching something in one room, and pick up exactly where I left off in another. I could look into moving the VM to the new host, but I also want to re-use the polestar name later, and I've never restored a mySQL database before.

I unfortunately had a hard time finding a truly fitting name for this new server. I'll be calling the new server highroad, from Badger Hill Brewery.

## Installation

Same story. CentOS 7.3, 1 CPU, 512-1024MB RAM, 32GB HDD. I pre-created the DHCP reservation for the new VM, and during the installation I set the host name and enabled the NIC, and the new VM was instantly added to DNS.

## MySQL Installation

I've never been a fan of the MySQL setup process, but I would rather migrate from MySQL to MySQL now and look into something like MariaDB later, so I'm hoping this is my last install process. The Kodi wiki has a [good page](http://kodi.wiki/view/MySQL/Setting_up_MySQL#tab=RedHat_based_Linux) detailing this process. I'll only deviate from that guide when it comes time to copy and migrate the database.

After logging in to the Oracle site so that I can get the RPM, I used SCP to copy it to highroad to install the repository.

```
rpm -i mysql57-community-release-el7-11.noarch.rpm
```

Then install MySQL community server.

```
yum install mysql-community-server -y
```

Enable and start mysqld.

```
[root@highroad ~]# systemctl enable mysqld
[root@highroad ~]# systemctl start mysqld
```

Open up the firewall.

```
[root@highroad ~]# firewall-cmd --add-service mysql
success
[root@highroad ~]# firewall-cmd --add-service mysql --permanent
success
```

## MySQL Setup

To log in to MySQL for the first time, we need the temporary password that was generated at startup.

```
[root@highroad ~]# grep 'temporary password' /var/log/mysqld.log
```

Now log in and change the password to something better.

```
mysql> SET PASSWORD for 'root'@'localhost' = PASSWORD('NewPasswordHere');
```

## Database Migration

First, we need to backup the old database. I don't have anything else running on this server, so I'm just going to back up the whole thing.

```
[root@polestar ~]# mysqldump --all-databases --single-transaction --user=root --password > all_databases.sql
```

Now I've got to get the database over to the new server.

```
[root@polestar ~]# scp all_databases.sql root@highroad:/root/
```

And restore the backup to the new install.

```
[root@highroad ~]# mysql -u root -p < all_databases.sql
```

## User Creation

I had an issue with the user account after the migration. I just simply dropped the migrated user and created a new one, since I didn't have anything fancy for permissions.

```
mysql> DROP USER 'kodi';
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE USER 'kodi' IDENTIFIED BY 'KodiPasswordHere';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL ON *.* TO 'kodi';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

## Test

I need to change my Kodi config to use the new server as the backend database now. If you're using this type of setup, the file will be in ~/.kodi/userdata/advancedsettings.xml

```
<advancedsettings>
    <videodatabase>
        <type>mysql</type>
        <host>172.16.0.71</host>
        <port>3306</port>
        <user>kodi</user>
        <pass>KodiPasswordHere</pass>
    </videodatabase>

    <musicdatabase>
        <type>mysql</type>
        <host>172.16.0.71</host>
        <port>3306</port>
        <user>kodi</user>
        <pass>KodiPasswordHere</pass>
    </musicdatabase>

     <videolibrary>
          <importwatchedstate>true</importwatchedstate>
     </videolibrary>
</advancedsettings>
```

Launch Kodi, and all my media shows up like before. Now I just need to change all the other Kodi clients in my house to use the new database server, and I can shut down polestar.

## Client Migrations

On ruination, the migration was pretty simple. The advancedsettings.xml file is located in %\AppData%\Kodi\userdata\advancedsettings.xml, and I made the above change to that file as well and opened Kodi to make sure it was connected.

The FireTV was a little more work. I needed to use the Android Debug Bridge (ADB) to connect and move files around. It doesn't need to be installed to run so I just called it from where I extracted the zip file. The full download is [here](https://developer.android.com/studio/releases/platform-tools.html#download).

First, I had to start the server and connect.

```
./adb kill-server
./adb start-server
./adb connect 172.16.0.110
```

Then I pulled the files that I make changes to, to back them up for the update (I'm also updating from 17.1 to 17.3), and to make the change to advancedsettings.xml.

```
./adb pull /sdcard/android/data/org.xbmc.kodi/files/.kodi/userdata/advancedsettings.xml
./adb pull /sdcard/android/data/org.xbmc.kodi/files/.kodi/userdata/passwords.xml
./adb pull /sdcard/android/data/org.xbmc.kodi/files/.kodi/userdata/sources.xml
```

At this point I made the planned changes to advancedsettings.xml (like above), and downloaded the new version of Kodi. Now it's time to update kodi. If this was a new install I would exclude the -r flag.

```
./adb install -r kodi-17.3-Krypton-armeabi-v7a.apk
```

And now time to push back the custom settings. I didn't bother checking if they needed to be changed or not and just pushed them.

```
./adb push advancedsettings.xml /sdcard/android/data/org.xbmc.kodi/files/.kodi/userdata/advancedsettings.xml
./adb push passwords.xml /sdcard/android/data/org.xbmc.kodi/files/.kodi/userdata/passwords.xml
./adb push sources.xml /sdcard/android/data/org.xbmc.kodi/files/.kodi/userdata/sources.xml
```

And that does it. I launched Kodi on the FireTV and again confirmed the database was connected to the new server (I marked certain files as watched on the new database and unwatched on the old database previously to make confirmation easy).

## Conclusion

A quick one here, but like the rest, an important step in moving over to the new infrastructure. I've been putting off migrating FreeNAS because once I pull out the existing authentication, all the media computers using Kodi around the house will stop working until I get it fixed. So far I've kept all the disruptions to a minimum, and I've been able to work on them as I have time since nothing stopped working during the migration. I'm getting close now to getting held up by the FreeNAS migration, so unless I do that next, I'll likely be doing some smaller projects like getting a new LibreNMS server set up. We'll see what time allows in the next few weeks.
