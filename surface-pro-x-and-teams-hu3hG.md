---
author: steve@getrubix.com
date: Mon, 23 Dec 2019 19:48:00 +0000
description: '"The Surface is great, Windows 10 is great and Office 365 is great.
  But the problem is that sometimes the folks who engineer these great products out
  in Redmond forget to speak to each other. If you’re looking to deploy a shiny new
  Pro X with Autopilot, then"'
slug: surface-pro-x-and-teams-hu3hG
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/surface-pro-x-and-teams-hu3hG_thumbnail.jpg
title: Surface Pro X and Teams
---

The Surface is great, Windows 10 is great and Office 365 is great. But the problem is that sometimes the folks who engineer these great products out in Redmond forget to speak to each other. If you’re looking to deploy a shiny new Pro X with Autopilot, then you’re going to find out the hard way that users are going to be asking you why Teams is missing. But don’t worry, my team of nerds…_I mean engineers_, have cracked this one.

![71WPXY5GhgL._AC_SX466_.jpg](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581104762006-HP2RM40QVEYN2LPC7S0Y/71WPXY5GhgL._AC_SX466_.jpg)

What’s the problem?
-------------------

Microsoft’s new hardware lineup consists of the Surface Laptop 3, Surface Pro 7 and Surface Pro X- all shipping out of the box with Windows build 1903 (specifically 10.0.18362). These PCs also have the complete Office 365 apps already installed. This is generally a good thing as that is one of the larger software deployments that can now be skipped through Autopilot and MEM (Microsoft Endpoint Manager). Only snag is that while the Pro 7 and Laptop 3 have Teams installed, the Pro X does not. To make matters worse, the \*32-bit version of Teams does not install correctly when pushed through MEM.

_\*Remember, the Pro X is an ARM based device that can emulate x86 bit applications. So that means no 64-bit apps here, just 32-bit._

…so why can’t we just install Teams?
------------------------------------

After Jesse and I banged our heads against the wall getting failure after failure when deploying Teams to the Pro X via MEM, my technical director pointed out a key piece of information about the way Teams installs. He’s way better at googling than we are.

You see, the standard behavior for Teams when Office is pre-installed is to sit quietly in **“C:\\Program Files (x86)\\Teams Installer”**. When the user logs in for the first time, the Teams .exe inside that folder runs for the first time and then performs the actual installation. That’s apparently just standard behavior for Windows 10 and Teams.

So now armed with knowledge of that behavior, I took all three brand new Surface devices out of their box and went searching through the explorer. And there it was- both the Laptop 3 and Pro 7 had the Teams installer sitting where it was supposed to be. The Pro X did not. The answer was clear. All we had to do was push the Teams install directory through MEM to where it was supposed to be before the user signs in to the device while it’s being provisioned with Autopilot- easy right?

![2020-02-07 14_44_30-Blog Feed – Inconvenient Integration.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581104787505-RMK8DUBPRH6CCR1FAPOL/2020-02-07+14_44_30-Blog+Feed+%E2%80%93+Inconvenient+Integration.png)

That doesn’t sound easy…
------------------------

There are a few assumptions you must understand in order to see the flow of what had to be done. If you’re not familiar, here is the break down:

-   The ESP (Enrollment Status Page) in MEM is set to block the user from logging in until Autopilot finishes provisioning device level apps. In this case (and most others) we are also skipping completely over the User portion of the ESP due to general wonkiness
    
-   Pushing through standard MEM application deployments would attempt to install it before the user object is created. That doesn’t work as the default install location for Teams is **“C:\\Users\\<USERNAME>\\AppData\\Local\\Microsoft\\Teams”** which doesn’t exist yet in the process
    
-   We heavily leverage Michael Niehaus’s [AutopilotBranding](https://github.com/mtniehaus/AutopilotBranding) package to basically run an assortment of staging tasks to the device during the initial device provisioning. If you’re not familiar, go check it out right now!
    

So we needed the Teams install directory to sit in **“C:\\Program Files (x86)”** before the user object is created. This means our only shot was to script it in to the AutopilotBranding PowerShell script.

The good part is simply launching the Teams MSI installer just places the correct install directory where it is supposed to be without trying to run the embedded .exe file. Here’s the flow for the PowerShell script block:

-   Check if Teams install directory exists first
    
-   If not, download directly from the cdn url
    
-   Quietly launch the .MSI, thus placing the directory in the right location
    
-   Do nothing- an important last step. Again, we do not want to run that .exe inside.
    

So knowing all of that good info, here is the script block that was added to the _AutopilotBranding_ packge:

if(!(Test-Path "C:\\Program Files (x86)\\Teams Installer")) {
Write-Host "Downloading Teams Installer MSI" 
$dest = "$($env:TEMP)\\Teams\_windows.msi" 
$client = new-object System.Net.WebClient 
$client.DownloadFile("https://teams.microsoft.com/downloads/desktopurl?env=production&plat=windows&managedInstaller=true&download=true", $dest)        
Write-Host "Installing: $dest"  
msiexec /i $dest /qn
} 

Simple right? Let’s break it down.

So the first thing we’ll do is test to see if the installer exists.

if(!(Test-Path "C:\\Program Files (x86)\\Teams Installer"))

You never know what could happen with Microsoft- they may start shipping an updated Pro X that has the installer and then we won’t need it. Also, throwing this in your _AutopilotBranding_ package means you don’t need different packages for each model. If this were run on a Pro 7 or Laptop 3, you’ll be fine since the installer is already there.

In the next step, we’re downloading the Teams MSI directly from the Microsoft CDN and placing it in the TEMP directory:

$dest = "$($env:TEMP)\\Teams\_windows.msi" 
$client = new-object System.Net.WebClient 
$client.DownloadFile("https://teams.microsoft.com/downloads/desktopurl?env=production&plat=windows&managedInstaller=true&download=true", $dest)

And finally, we run the MSI:

msiexec /i $dest /qn

That’s it! If everything goes to plan, you should see Teams installing as soon as the user logs in.

![2019-12-23-10_59_24-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581104930793-DEU4ZIJ0R7G2VEEYMLGI/2019-12-23-10_59_24-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png)
