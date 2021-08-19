---
title: Profiles, Bashrc, and Zshrc
date: 2021-08-19 11:29
categories: [General]
author: Bwilliamson
tags: [windows linux osx profiles bashrc zshrc powershell bash]
---

# Let's talk about profiles.  

Something I don't see used often enough is profiles. Sysadmins, Support, and even Developers/Engineers that I've worked with would give me a raised eyebrow when I would ask if they have that script/command/alias in their profile yet.

If we're on Windows - I'm talking about your ***`$Profile`***, if we're on OSX/Linux I'm talking about your ***`~/.zshrc`*** or ***`~/.bashrc`*** depending on your version. I won't get into the weeds with ***`/etc/profile.d`*** here, and just stick to the basic profiles. 

Profiles, are simple scripts that are run with each new terminal/powershell session, depending on the scope. In windows there can be multiple profiles depending on your scope, which I'll touch on in a moment. In OSX/Linux we're generally using the profile of the current user. This allows us to create a lot of time-saving code snippets, aliases, and functions that we use during our day.

## $Profile (Windows)

Something I love to set up on my windows workstations is plenty of handy commands, functions, and aliases. Windows Powershell has a lovely way of interacting with the system via it's **verb-noun** syntax. E.g. ***`get-vm`***, ***`get-content` (cat/gc)***, ***`set-content`***, ***`get-childitem` (ls/gci/dir)***, etc. This makes it pretty intuitive for using the built in commandlets. 

### Scope and how it relates to the what/where 

*Activity*: Open a powershell prompt and type `$profile` and press enter.  
Now type `$profile | fl -force` and press enter.  
What the heck are all these paths?
```powershell
PS C:\WINDOWS\system32> $profile
C:\Users\bwilliamson\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
PS C:\WINDOWS\system32> $profile | fl -force


AllUsersAllHosts       : C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1
AllUsersCurrentHost    : C:\Windows\System32\WindowsPowerShell\v1.0\Microsoft.PowerShell_profile.ps1
CurrentUserAllHosts    : C:\Users\bwilliamson\Documents\WindowsPowerShell\profile.ps1
CurrentUserCurrentHost : C:\Users\bwilliamson\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```

The **TLDR** here is that windows has a lot of different scopes for your profiles. Generally the only one we need to be concerned with is our default `$profile` as this works in both regular, and elevated powershell sessions. If we're doing any work with PS ISE, then that can use yet another set of profiles which I'll skip for brevity. 

### Usage and Examples

Let's put this to work for us. In windows I've had a lot of fun building silly functions as well as usefull ones. Feel free to crawl my [powershell script repository](https://github.com/Bwilliamson55/Powershell-and-batch-scripts) - it's a bit out of date, but the functionality should remain. 

Here's a quick and dirty how-to on profiles from that repo:  
```powershell
#In order to have certain modules built into the powershell session, without remembering the command for them, add them to the powershell profile on the machine.
#There are multiple profiles for each machine and their info can be viewed with the following:
$profile | get-member -type noteproperty | FL Name, Definition
#There isn't always a profile. So to create one, without overwriting any existing use this command in a PowerShell-
if (!(test-path $profile))             {new-item -type file -path $profile -force}
#The output of this command will show where and if a profile was created.
```

Here's a commom function I include in my profiles, these days-  
```powershell
function View-Hosts {
notepad $env:windir/system32/drivers/etc/hosts
}
```

How many times have you gone digging for that silly file? No more! Drop this in you `$profile` and start a new powershell session. You'll now have a handy new alias to bring up the hosts file. Just be sure to run that function from an admin session.

Here's some more useful `$profile` stuff I use  
```powershell
#$profile
$sysfunctions = gci function:
#my functions here -----
$psdir="C:\Users\bwilliamson\Documents\WindowsPowerShell\Scripts"  
# load all 'autoload' scripts
Get-ChildItem "${psdir}\*.ps1" | %{.$_ ; write-host "$_.name Loaded"} 
import-module activedirectory
Import-Module PowerShell-Beautifier.psd1
Write-host "Active-Directory Cmdlets loaded"
Write-Host "Custom PowerShell Environment Loaded" -fore yellow
Write-Host "Custom Functions Loaded:" -fore green
gci function: | where {$sysfunctions -notcontains $_}
# Chocolatey profile
$ChocolateyProfile = "$env:ChocolateyInstall\helpers\chocolateyProfile.psm1"
if (Test-Path($ChocolateyProfile)) {
  Import-Module "$ChocolateyProfile"
}
```

As a last note- you can also create your own aliases here via `New-Alias AliasName Cmdlet`  
If you want to make an alias to a command with arguments, you must use functions in your profile instead.

## ~/.zshrc and ~/.bashrc

On OSX and Linux we use bash scripts for our profiles. These can be found in the home directory, and depending on your version of OSX/Linux and your preferred teminal type, can change a bit in name. Generally though, if you're on a new Mac (Big Sur for instance) you'll be using ~/.zshrc, if you're on debian 9, or ubuntu let's say, then you'll probably be looking for ~/.bashrc.

The usage between these two is identical. You can write as many aliases or imports as you want. 

### Usage and Examples

This is much easier than windows, in that we can simply type what we want our aliases to output into the termainal. Commands and bash functionality alike. We can also use functions to validate input on multiple parameters- more on that shortly.

Once we put aliases and functions into our profile, we just need to source it in our terminal (`source ~/.zshrc`), or start a new terminal session.

I do mostly web development these days, largely around the Magento framework. Due to that, my profile is a bit tame, but it will serve as a good example for this post. 

```bash
#set some globals
export EDITOR=nano
export VISUAL="$EDITOR"

#ElasticSearch and Mysql
#($project_name here is for my vagrant scripts, if you have a consistent environment then change those out)
alias check-elastic='curl -XGET "localhost:9200/_cluster/health?pretty"'
alias mysql-root='mysql -h 127.0.0.1 -u root'
alias mysqldump-magento="mysqldump ${PROJECT_NAME} -u root -p -h 127.0.0.1 | gzip > ~/dbdump/${PROJECT_NAME}_$(date "+%Y-%m-%d_%H%M").sql.gz"

#zcat a gzip sql file into specified db
#e.g. restore-db ~/dbdump/myfile.sql.gz magentoDBname
#for OSX use gzcat - which takes the same arguments
function restore-db() {
    dbfile=${1:?"The path must be specified."}
    dbname=${2:?"The destination db name must be specified."}   
    zcat $dbfile | mysql -h 127.0.0.1 -u root -p $dbname
}

##Removing things
alias flush-static='rm -rf pub/static/* ||: && rm -rf var/view_preprocessed/* ||: && rm -rf var/cache/* ||: && rm -rf var/page_cache/* ||:'
#||: here is to be sure it continues if the directory is not found or already empty.
#I have many more aliases for flushing stuff but I've removed them for brevity

##Compile and Deploy things
alias compile-mage='bin/magento set:up && php -dmemory_limit=4G bin/magento setup:di:compile && bin/magento c:c'
#etc

##Aliases for Aliases
alias set:up='bin/magento set:up'
alias di:compile='php -d memory_limit=4G bin/magento setup:di:compile'

#Git things
alias fpg='git fetch && git pull'
```

## Conclusion

Profiles are cool. Use them. Stop typing stuff like `bin/magento` three million times a day. Your fingers will thank you.