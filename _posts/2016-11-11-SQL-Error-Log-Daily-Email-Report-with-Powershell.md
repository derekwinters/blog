---
layout: post
title: SQL Error Log Daily Email Report with PowerShell
date: 2016-11-11 19:00:00 -0600
categories: powershell sql scripting
---

# SQL Error Log Daily Email Report with PowerShell



In my job, I wind up helping out our DBAs with various tasks for setup, troubleshooting, and reporting. It’s definitely not my main job and I’m still learning a lot from them and from training courses (SQL Skills Accidental DBA was a great one), but when they told me they had some simple PowerShell they were using manually on a daily basis, I jumped at the chance to dig in more and get that automated for them.

One of the scripts they had would check the SQL error log, filter out unwanted errors, and display the results in a table. I took that script, added some logging, and turned it into a daily scheduled task hosted on of our utility servers that runs other scheduled scripts via the Windows Task Scheduler. It wound up taking a task that could take 30+ minutes each morning to a quick 5 minute skim through emails to see if anything needed to be investigated further.

```
<#
	.SYNOPSIS
	
	Find errors in SQL Error Log file that occurred in the past $hours, default is 24 hours.

	.DESCRIPTION

	Returns the list of errors that match a list of known errors in the past $hours on the target SQL server. Optionally the results can be sent via email, if errors matching the list are found.

	.EXAMPLE
	
	.\Get-SqlErrorLog.ps1 -SQLServer SqlServerHostname -EmailTo "user@example.com" -Hours 24 -SmtpServer "smtp.example.com"
	
	#>
################################################################################
################################################################################
##
## Parameters
##
################################################################################
################################################################################
[ CmdletBinding ( ) ]
	Param (
		[ Parameter ( Mandatory = $true ) ]
			## SQL Server
			[ string ]
			$SQLServer,
		[ Parameter ( ) ]
			## Email to
			[ string ]
			$EmailTo,
		[ Parameter ( ) ]
			## Hours
			[ int ]
			$Hours = 24,
		[ Parameter ( ) ]
			## SMTP Server
			[ string ]
			$SmtpServer = "smtp.example.com"
	)
################################################################################
################################################################################
##
## Preparation
##
################################################################################
################################################################################
## Required module, can be imported automatically
#import-module SQLServer

## Log the start
Write-EventLog -LogName Application -Source "Get-SQLErrorLog" -EntryType Information -EventId 1000 -Message "Beginning run against $($SQLServer)."

## Test Server connectivity
If ( !( Test-Connection -ComputerName $SQLServer -Quiet -Count 1 ) ) {
	$Message = "Could not connect to $($SQLServer)."
	Write-EventLog -LogName Application -Source "Get-SQLErrorLog" -EntryType Error -EventId 1001 -Message $Message
	Throw $Message
}

################################################################################
################################################################################
##
## Retrieve Error Log
##
################################################################################
################################################################################
$ErrorLog = @( Get-SqlErrorLog -ServerInstance $SQLServer -ErrorAction Stop | Where-Object { `
	( $_.text -like "*Error*"  -or $_.text -like "*Fail*" -or $_.text -like "*dump*" -or $_.text -like "*IO requests taking longer*"  -or $_.text -like "*is full*"  )`
	-and ( $_.text  -notlike "*found 0 errors*" ) -and ( $_.text  -notlike "*without errors*" ) -and ( $_.Date -ge ( ( Get-Date ).AddHours( -$( $hours ) ) ) ) } | Sort-Object Date -descending )

################################################################################
################################################################################
##
## Email Error Log
##
################################################################################
################################################################################
## Output
$Message = "Found $($ErrorLog.Count) errors on $($SQLServer)."
Write-Host $Message
Write-EventLog -LogName Application -Source "Get-SQLErrorLog" -EntryType Information -EventId 1002 -Message $Message

## Send email if desired
If ( $ErrorLog.Count -gt 0 -and $EmailTo ) {
	## Build the email body
		$Body  = "<html>`n"
		$Body += "  <body>`n"
		$Body += "    <br />`n"
		$Body += "    <table border=1>`n"
	ForEach ( $Line in $ErrorLog ) {
		$Body += "      <tr>`n"
		$Body += "        <td width=100> $($Line.date) </td>`n"
		$Body += "        <td> $($Line.Text) </td>`n"
		$Body += "      </tr>`n" }
		$Body += "    </table>"
		$Body += "</body>"
		$Body += "</html>"
	## Send the email
	$From = "$($env:computername)@example.com"
	Write-EventLog -LogName Application -Source "Get-SQLErrorLog" -EntryType Information -EventId 1003 -Message "Sending email from $($from) to $($EmailTo) via $($SmtpServer) for server $($SQLServer)."
	Send-MailMessage -To $EmailTo -From $From -Subject "There are $($ErrorLog.Count) errors in the $($SQLServer) error log" -BodyAsHtml -Body ($Body) -SmtpServer $SmtpServer
}

$ErrorLog
```

The logic is pretty straight forward, but here are the highlights. Everything important has a parameter so it can be configured to run as frequently as necessary without code changes.

First, we load what’s needed. Mainly, the SQLServer PowerShell module. It’s included in SQL Server Management Studio 2016, and can be installed by itself. We have it installed in a default module location, so we no longer need to manually import it. After that, we log the start of the script and test for connectivity to the server before continuing. One thing that I don’t do within the script is make sure the Event Log source exists. I might add it, but right now I just run it before running the script for the first time on a server. It’s a simple one-line PowerShell command. [Technet: New-EventLog](https://technet.microsoft.com/en-us/library/hh849768.aspx).

```
New-EventLog -LogName Application -Source "Get-SQLErrorLog"
```

The next step is to get the error log, and filter out what we don’t care to be alerted to. Right now I’m using a very big Where-Object to filter, but I’m planning to try some array comparisons to make it easier to manage in the future.

The final step is the email, if errors were detected. There is no reason to get an email that no errors were detected, except to ensure the script is still running. That is what the event logging is for though, and you can set up monitoring of the event log for errors if you wanted. With 30 servers, we really don’t want to get emails for each one unless we need to.

I am using the old table HTML for the email body. It’s old and doesn’t do anything pretty, but we didn’t need it to be pretty. All we wanted was the errors and the timestamps so we could determine if we needed to investigate further. I’d like to add an additional column to the email table that identifies possible causes for the error messages. One error we see on occasion is related to an issue with a backup using a third party tool. The error message isn’t very descriptive so it would be nice to have one or two possible causes listed with it, and even a link to our documentation on how to resolve the issue.

There are a lot of other valid ways to do this, but for us, this was a simple way to get the errors to us each morning to keep an eye on all our servers.
