---
title: Extract Exchange message tracking logs with powershell
date: 2021-08-22 19:17
categories: [Sysadmin, Powershell]
author: bwilliamson
tags: [powershell, sysadmin, etl, exchange, csv]
---

Exchange makes it kind of difficult to do complicated reporting without third party tools. This post will show how to extract the message logs useful data.

For the purpose of this post, we are not actually on the exchange machine - we are using a separate windows VM that is used for reporting purposes. This windows VM is a domain member.

# Setup

We already have a user account set up on the domain with just enough privilege to pull this log. But, to be on the safe side we're also going to generate a securestring text file with our password, as this task runs unattended and there may be other users with wandering eyes.

You should use best practice of _least privilege necessary_ but I'm not your boss, use your domain admin if you're feeling brave. ***(Please don't)***

## Generate a secure string file for password storage

```powershell
read-host -assecurestring | convertfrom-securestring | out-file C:\temp\mysecurestring.txt
```
_Swap out the filepath as needed_

The main idea here is to obscure the password from other admins and users that may have access to the reporting VM. It is not a bullet proof security measure.

# TLDR

For those that just want to grab the script, here it is. Keep in mind there are many date methods you can use for your start/end times such as:
* AddDays
* AddHours
* AddMilliseconds
* AddMinutes
* AddMonths
* AddSeconds
* AddTicks
* AddYears



## The script - currently set to grab the past day

Swap out the usernames, file directories, machine names, etc for your environment.
```powershell
###############
#
# Exchange Export Message Tracking Log
#
# The purpose here is to export the message tracking log (With only useful fields).
# End date should be no sooner than last night at 23:59:59, and start date should always be at 00:00:00
# This will keep the data sane.
# We are trimming the file size by filtering:
# One source, and two events. STOREDRIVER + RECEIVE = A mail sent. STOREDRIVER + DELIVER = A mail received
# This is not intuitive, thus:
# More info: https://docs.microsoft.com/en-us/powershell/module/exchange/get-messagetrackinglog?view=exchange-ps
# https://docs.microsoft.com/en-us/exchange/mail-flow/transport-logs/message-tracking?view=exchserver-2019
#
# Fields pulled:
#Timestamp
#ClientIp
#ClientHostname
#ServerIp
#ServerHostname
#ConnectorId
#Source
#EventId
#MessageId
#Recipients
#RecipientStatus
#RecipientCount
#RelatedRecipientAddress
#MessageSubject
#Sender
#ReturnPath
#Directionality
#OriginalClientIp
###############

#generate a secure password file, this is a one time one liner, left here for reference
#read-host -assecurestring | convertfrom-securestring | out-file C:\temp\mysecurestring.txt

$username = "DOMAIN\USERNAME"
$password = Get-Content 'C:\Reports\Powershell reports\mysecurestring.txt' | ConvertTo-SecureString
$cred = new-object -typename System.Management.Automation.PSCredential -argumentlist $username, $password

#Connect to the Exchange Server using the secure cred above
$Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri http://MyExchangeHostname.domain.local/PowerShell/ -Authentication Kerberos -Credential $cred
Import-PSSession $Session
write-host "Assigning Variables..."

#Sometimes I use dates or times in my filename, currently they are unused and here as an example.
$today = (get-date -Format M-d-yy)
$timestamp = (get-date -Format HHmmtt)
$outfilepath = "C:\Reports\Powershell reports\"
$outfilename = "EmailTrackingLogPastDay.csv"
$theDate=Get-Date

#Subtract days to establish start and end times. Can also use months/years etc
$startDateFormatted=$theDate.AddDays(-2).ToShortDateString()
$endDateFormatted=$theDate.AddDays(-1).ToShortDateString()

#keep the cmdlet readable by putting the field list into this variable
$fields = "Timestamp", "ClientIp", "ClientHostname", "ServerIp", "ServerHostname", "ConnectorId", "Source", "EventId", "MessageId", "Recipients", "RecipientStatus", "RecipientCount", "RelatedRecipientAddress", "MessageSubject", "Sender", "ReturnPath", "Directionality", "OriginalClientIp"

#Get the tracking log and export to a CSV.
#To trim the file- we are using one source, and two events. STOREDRIVER + RECEIVE = A mail sent. STOREDRIVER + DELIVER = A mail received
write-host "Reading Exchange Log"
Get-MessageTrackingLog -Start "$startDateFormatted 00:00:00" -End "$endDateFormatted 23:59:59" -ResultSize Unlimited | Select $fields | ? {($_.Source -eq "STOREDRIVER") -and ($_.EventId -eq "RECEIVE" -or $_.EventId -eq "DELIVER")} | Export-CSV "$outfilepath$outfilename" -NoType

#Dont forget to disconnect with..
Remove-pssession $session
```
Depending on the exchange server size this log can get heavy. Keep that in mind while this runs- it takes some time if you're looking at months/years.

**Note**: Default Exchange server MTL retention is only **30 days**. I will expand this post in the future if I'm asked, to include how to extend that.

# Automate it

The easiest way to automate this script will be with Windows Task Scheduler.

The screenshots are mostly self explanatory:
1. Create the task, name it, configure the security options
     1. The user is not of great importance in this case as our script has built in credentials.
2. Under the Triggers tab- set your schedule per your script (Daily monthly etc)
3. The Actions tab - Start a program, script location, and arguments (Details after the screen shots)

![Task Creation Screen](/assets/img/post images/sysadmin/Exchange tracking log task/01.png){: .normal }
01

![Task Triggers](/assets/img/post images/sysadmin/Exchange tracking log task/02.png){: .normal }
02

![Task Actions](/assets/img/post images/sysadmin/Exchange tracking log task/03.png){: .normal }
03

The action part of the task is always tricky with powershell.
For this example, even though it may not be best practice, we are using bypass.

So run the program script field:

`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

Arguments field:

`-NoProfile -NoLogo -NonInteractive -ExecutionPolicy Bypass -File "C:\Reports\path bla bla\Awesome_Email_Log_Script.ps1"`

# Conclusion

Not that difficult right? Just wait until you want to parse the data..

I highly recommend reading the MS docs on the tracking log, specifically interacting through powershell, and play around with the different results you can generate.

I will be cross-linking this post to my post about dumping this data into mysql, once I work up that post.
