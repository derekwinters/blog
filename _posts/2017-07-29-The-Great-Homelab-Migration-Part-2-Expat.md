---
layout: post
title: The Great Homelab Migration - Part 2 - Expat
date: 2017-07-29 19:00:00 -0600
author: Derek Winters
categories: homelab
---

The NUC has arrived and it's time for the first build and migration. It's a NUC, so there really isn't a whole lot to the build though so this will mostly be about what comes after the build.

I did come up with a new name for this build. I was going through a few beer lists for some local breweries looking for some good names (I have around 30 now, should cover me for a while) and I came across one that I liked a lot more than chainbreaker for the new NUC running Windows in a world of desktops and Linux. This build will be named Expat, after the beer from Fulton Brewery.

## The Hardware

[Intel NUC NUC5CPYH](https://www.amazon.com/gp/product/B00XPVRR5M/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)

[Kingston Digital 120GB](https://www.amazon.com/gp/product/B01FJ4UN76/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)

[Crucial 8GB Single DDR3L](https://www.amazon.com/gp/product/B006YG8X9Y/ref=oh_aui_detailpage_o00_s01?ie=UTF8&psc=1)


## OS Install

Since I don't have any of the other infrastructure rebuild yet, I'll be using my existing Windows DHCP server for now, and will transfer over later when the new system is up. I won't be joining my domain either.

For the first time in a while, I ran into multiple issues during the Windows install. I guess this post needed some more content.

The first issue was the bootable USB. I have an 8GB USB stick that hosts whatever operating system was last installed. In this case it was Fedora, and needed to be re-imaged. I tried Multiwriter and Fedora Media Writer, but neither was recognized by the NUC as a bootable image, trying two different ISOs that I have. Hoping to get at least the imaging finished on the first night, I went back to my Windows install on my desktop and used Rukus. That worked and the installer loaded so I rebooted back to Fedora, but not before Windows forced an update.

Once I got back into Fedora I looked over to the NUC monitor, and the install failed for an unknown reason. It's been a while since I used burned any of these images, so it was time to try another image, and back into Windows. Turns out both of my April 2016 images have issues, so it's back to the original release image of Windows 10, and lots of updates are coming my way.

### Customizations

I have a number of small group policies on my domain that I'll need to make manually now that the new computer won't be joined to the domain.

#### Remote Management

Enabling remote access is simple. System Properties > Remote > Remote Desktop > "Allow remote connections to this computer."

#### Power Management

I don't want this computer to ever go to sleep or stop any services.

Control Panel > Hardware and Sound > Power Options

- I don't want any button presses to do anything. Set "Do nothing" for all power button settings.

I'm going to create a new power plan called HomeSeer, based on the High Performance power plan. In the basic setup I set never turn off display, never put computer to sleep. Once the plan is created, I'll change plan settings to set the advanced settings.

- Hard Drive
  - Turn off hard disk after
    - Never

I never plan on using WiFi or Bluetooth on this system either, so I disabled both of those interfaces. If I change my mind I can always turn them back on.

#### SNMP

I want to monitor this system with SNMP. This isn't installed by default, so open "Turn Windows features on or off" and select everything under "SNMP" to install the service. After the install, go to "Services" and select "Properties" in the SNMP service to set the community string. I'm not too concerned with security at this point, so I allow SNMP polling from any host as long as they have the community string.

#### Firewall

Besides the HomeSeer ports which should be enabled during the installation, I also want to make sure Remote Desktop and SNMP are enabled. Turns out they are both enabled when you allow RDP and install SNMP.

## HomeSeer Migration

Next up, reinstalling and migrating my existing HomeSeer setup to the new computer.

### Installation

The HomeSeer installer (I have the installer from 3.0.0.181 so I'll have some updating to do afterwards), doesn't ask for a whole lot. It installs some Visual Studio requirements, and asks for a location for the program files. Before applying my license or restoring the backups, I launched HS3 to get the updates finished so I'm restoring to the same version. Turns out my existing version was a little out of date too, so I updated that as well. Both are now on 3.0.0.318 and I'm ready to continue.

### Finalizing the Initial Setup

There aren't awesome backup and restore methods for HomeSeer. Backing up the entire Program Files folder is the recommended way to go, and that's what I will do.

The first thing I did was backup the Z-Wave controller through the web interface prior to shutting down the server. This is done through `Plugins > Z-Wave > Controller Management`. From there, select the **backup this interface** option, and run the backup.

I have a schedule task that runs a PowerShell script to manage automatic backups for HomeSeer. It grabs a copy of a few directories and places them in a staging folder. Once there, it creates a zip archive with the date and time, and copies the archive to my backup share. It then sends an email to MMS to my phone with the status. If it errors, out, it should also catch and send me a failure notification.

```
################################################################################
##
## Send an email in the event of a failure
##
################################################################################

Function Send-BackupResult ([switch]$success) {

    $emailSmtpServer         = "smtp.gmail.com"
    $emailSmtpServerPort     = "587"
    $emailSmtpUser           = "emailAddressForHouse@example.com"
    $emailSmtpPass           = "AccountPassword"
    $emailMessage            = New-Object System.Net.Mail.MailMessage
    $emailMessage.From       = "emailAddressForHouse@example.com"
    $emailMessage.To.Add("123456789@cell.phone.carrier.emailToMMS")
    $emailMessage.Subject    = "HomeSeer Backup"
    $emailMessage.IsBodyHtml = $true

    If ( $success ) {
        $Result = "succeeded"
    }
    Else {
        $Result = "failed"
    }

    $emailMessage.Body       = @"

The backup for HomeSeer $($Result).


$(Get-Date -Format D) $(Get-Date -Format t)
"@



    $SMTPClient = New-Object System.Net.Mail.SmtpClient( $emailSmtpServer , $emailSmtpServerPort )
    $SMTPClient.EnableSsl = $true
    $SMTPClient.Credentials = New-Object System.Net.NetworkCredential( $emailSmtpUser , $emailSmtpPass );

    $SMTPClient.Send( $emailMessage )

}

################################################################################
##
## Backup Function
##
################################################################################
Function Backup-HSDirectory {
    [CmdletBinding()]
        Param(
            [Parameter()]
                ## Source Directory
                [string]$Source,
            [Parameter()]
                ## Name of backup
                [string]$Name,
            [Parameter()]
                ## Staging Directory
                [string]$Staging = "C:\Staging",
            [Parameter()]
                ## Destination Directory
                [string]$Destination = "\\Remote\Folder\For\Backups"
        )

    ############################################################################
    ## Setup
    ############################################################################
    ## Date string
    $Date = Get-date -Format "yyyyMMdd.HHmmssffff"

    ############################################################################
    ## Pre-Validation
    ############################################################################
    Try {
        ## Ensure we can read the source directory
        Test-Path -Path $Source -ErrorAction Stop | Out-Null

        ## Ensure the temporary location exists and we have some free space
        Test-Path -Path $Staging -ErrorAction Stop | Out-Null
    
        ## Remove anything in the staging folder
        $Files = Get-ChildItem -Path $Staging
        If ( $Files ) {
            ForEach ( $File in $files ) {
                Remove-Item $File.FullName -ErrorAction Stop
            }
        }

        ## Ensure we can write to the final location
        Test-Path -Path $Destination -ErrorAction Stop | Out-Null
    
        $Guid = New-Guid
        $TempFile = "$($Destination)\$($Guid).txt"

        Add-Content -Path $TempFile -Value "testing read/write" -ErrorAction Stop

        Remove-Item -Path $TempFile

        ## Save the folder name
        $StagingFolder = "$($Staging)\$((Get-Item -Path $Source).Name)"

        ## Backup Name
        $BackupName = "$($Staging)\backup-$($Name)-$($Date).zip"
        }
    Catch {
        Throw "Error during pre-validation"
    }
    ############################################################################
    ## Backup Actions
    ############################################################################
    Try {
        ## Copy the directory locally for staging
        Write-Host "Copying source to staging location"
        Copy-Item -Path $Source -Destination $Staging -Recurse -ErrorAction Stop

        ## Zip the directory and give it a new name
        Write-Host "Creating archive"
        Add-Type -AssemblyName "system.io.compression.filesystem"
        [io.compression.zipfile]::CreateFromDirectory($StagingFolder, $BackupName)

        ## Cleanup the folder
        Write-Host "Cleaning up staging folder"
        Remove-Item $StagingFolder -Recurse -ErrorAction Stop -Force

        ## Move the backup
        Write-Host "Moving backup file to destination"
        Move-Item -Path $BackupName -Destination $Destination -ErrorAction Stop
    }
    Catch {
        ## Cleanup the folder
        If ( Test-Path -Path $StagingFolder ) {
            Remove-Item $StagingFolder -Recurse -Force
        }

        If ( Test-Path -Path $BackupName ) {
            Remove-Item $BackupName -Recurse -Force
        }

        ## Exit function
        Throw "Error during backup"
    }

}

################################################################################
################################################################################
##
## Script Execution
##
################################################################################
################################################################################
## Run all backups. If any fail, report failure and stop.
Try {
    ## Backup HomeSeer directory
    Backup-HSDirectory -Source "C:\Program Files (x86)\HomeSeer HS3" -Name "HomeSeer" -ErrorAction Stop

    Backup-HSDirectory -Source "C:\Users\dwinters\Documents\HSTouch" -Name "HSTouchFiles" -ErrorAction Stop

    Backup-HSDirectory -Source "C:\Users\dwinters\Documents\My HSTouch Projects" -Name "HSTouchProjects" -ErrorAction Stop

    Send-BackupResult -success:$true
}
Catch {
    $Error[0]
    Send-BackupResult -success:$false
}
```

Next up, I installed HSTouch Designer, the app designer for their mobile and desktop app. I only have an Android mobile app right now, but I hope to make another for a tablet or desktop interface next. I'm hoping to make another post detailing my whole HomeSeer setup and how the app works soon.

#### Scheduled Tasks

##### Automatic Startup

In my existing setup, I have a scheduled task that runs on system startup that starts HomeSeer. I haven't been able to find a better way to manage automatically starting HomeSeer, so I'll transfer this over as well. The only downside to this task is it makes it more difficult to stop the service.

I like to keep my tasks organized and in their own folders, so I make a HomeSeer folder first. After that, I start the "Create New Task" dialog. Give it a name and description, select "Run whether user is logged on or not", "Run with highest privileges," and change to a Windows 10 task.

The trigger is "At system startup" and the action is Start a Program: "C:\Program Files (x86)\HomeSeer HS3\HS3.exe".

EDIT: I ran into an issue after the auto-start task ran. Scripts were failing to run, claiming "Missing Scheduler.dll" in the error message. After looking through the HomeSeer forums it turns out I also needed to set Start In to "C:\Program Files (x86)\HomeSeer HS3" to fix that issue. So far it has proven to fix the issue.

##### Backup

I need to make sure to get backups running again right away as well. I'll take the script from above and change it to save locally for now. Once I remove FreeNAS from the domain, I'll get some users set up and move the backups back to the NAS. I have my backups running every Sunday at 8:15PM. For the script to run, I need to enable scripts to run. Again, I'm not too concerned about security right now and will be letting anything run.

`Set-ExecutionPolicy -ExecutionPolicy Bypass`

The script doesn't create any of the base folders if they don't exist, so I create C:\Staging and C:\Backups before running the script to ensure it can access all the directories it needs and can write where it needs to. I now have a backup of the base install as well if I need to roll anything back later. I create another scheduled task, but this time the action is to start C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe with the path to the script as the argument.

##### External Access

I also have a few HomeSeer tasks that run scripts that write data to a share. I'll need to modify these once I restore from backup.

### Data Restore

It's time for the restore. I grabbed the latest backups from the FreeNAS share and extracted each archive. Replace the Program Files directory and the two Documents directories with the backups, and the first part is done.

#### Drivers

I use the HomeSeer SmartStick+. It has a driver provided that I'll install after connecting the USB stick. The initial driver install succeeds, and I open Device Manager to install the full driver.

#### HS Touch Designer

The Designer backups in the previous steps should have restored everything we need for HSTouch Designer, but we'll need to validate after the initial startup tests.

#### Validations

Time to start up HomeSeer and see if we were successful.

No errors, and looking through the Startup Log shows everything loading successfully. Time to test the Web Interface and mobile app.

All the devices show up and seem to have legitimate statuses. Moving over to the plugins, and everything is running as well.

And the final test is the mobile app. The myHS service sees the new server, sees that we are on the same network, and connects me properly to the new IP address for expat. A few minor changes in my events that are logging to FreeNAS and the migration is complete and a success. Let's backup our new server to be safe.

## Conclusion

That wraps up the catalyst project that kinds off everything else. It wasn't a huge project, but it was probably the system I care the most to keep running and backed up. I'm going to pull the hard drive out and save it for a few weeks in case of any issues. I've disabled the startup and backup tasks to make sure when the old server turns on, a broken HomeSeer install (because of the missing Z-Wave stick) doesn't start up. And I don't want to be writing bad backups to FreeNAS, even if the new ones aren't going there yet.

The new little NUC is chugging along and using a whole lot less power and space. It also now doesn't require my domain or FreeNAS, so as I go through the rest of my lab migration I know the automation and security I have set up will keep working.

When I have some time, I'll be going through the installation of the small hypervisor on the now free hardware from chainbreaker next. I originally was going to use oVirt, but after a lot more reading, I'm not sure I'll have enough hardware to satisfy the requirements. Wanting to stick with a Linux-based hypervisor, I may be looking at either KVM or Proxmox. We'll see where I land on that decision when the time comes.
