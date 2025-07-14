---
author: steve@getrubix.com
categories:
- intune
- powershell
- automation
date: Wed, 15 Jul 2020 04:27:36 +0000
description: >
  If you’ve deployed any Windows 10 devices via Autopilot, then you are no doubt familiar with the Enrollment Status Page (ESP). It is responsible for communicating which stage of the provisioning process that Windows is currently going through.
slug: please-wait
tags:
- endpoint manager
- script
- powershell
- intune
- flow
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/please-wait_thumbnail.jpg
title: Fighting the Enrollment Status Page
---

If you’ve deployed any Windows 10 devices via Autopilot, then you are no doubt familiar with the Enrollment Status Page (ESP). It is responsible for communicating which stage of the provisioning process that Windows is currently going through.

![esp-1903.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1594750659924-XRJ4197VAK7J3RN6FXQG/esp-1903.png)

This is a great feature as it allows IT admins to determine which applications must be installed on the machine prior to allowing a user to sign in. The only problem is, the ESP is quite unforgiving when installing these applications. Any hiccups that may occur during the installation will show themselves 100% of the time here, so it’s important to understand what can potentially trip up the ESP and what you can do to conquer it.

A giant clock
-------------

One of the more prominent features that can be configured in ESP is the amount of time to wait for provisioning to finish before an error message is thrown.

![2020-07-14 14_24_19-All users and all devices _ Properties - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1594751917577-ME6GUX4O10BQ6O030GBD/2020-07-14+14_24_19-All+users+and+all+devices+_+Properties+-+Microsoft+Endpoint+Manager+admin+center.png)

The default is 60 minutes, and unless you’re feeling especially bold, I don’t recommend changing it. We’ve given ourselves a preset amount of time for all of our required applications and policies to come down from Microsoft Endpoint Manager (MEM, formerly known as Intune). Granted, if you’re pushing more than an hour’s worth of content during this phase, the question is why are you using Autopilot to begin with? But still, even if your average provisioning time is in the 20-30 minute range, having some wiggle room for poor bandwidth is smart.

And while we’re on the subject of time, DO NOT configure anything that can mess with the PC time or Windows time service during this phase. This should be self explanatory, but given that ESP is actually just a timer, I wouldn’t recommend interfering with it.

Please wait…
------------

One of the more aggravating traits of the ESP is the way it checks if an application has installed successfully. Well technically this falls on the Intune Management Extension (IME… I know, I know; more acronyms). The flow is supposed to work as follows:

1.  ESP receives instructions of which apps it needs to install
    
2.  Intune management extension looks at the application ID and downloads the content
    
3.  Application install starts
    
4.  Intune management extension checks the configured detection rule to determine if that app has succeeded or failed.
    

What happens if step 3 fails on the first try, or it takes longer than it should to be successful? That means step 4 may occur too early and report a failed application, regardless of whether or not the app is actually installed. This hiccup is very common when your app installs are started via a script (batch file, PowerShell, etc.). Let’s have a look at one of those scripts:

```
Start-Process msiexec "$($PsScriptRoot)\applicationInstaller.msi" -ArgumentList "/i /qn"
```

Looks harmless, right? Well, if Intune is kicking off that script to install the application, then our work flow will probably end up looking like this:

-   Application install starts
    
-   PowerShell kicks off the configured script to run application
    
-   Intune Management Extension checks for the detection rule and determines if the installation succeeded or failed
    
-   PowerShell script is still running….
    

See the problem? I’ve seen this quite a bit, and it is extremely frustrating to see the ESP fail due to an app being reported as failing, especially when in reality it installed. The best way to solve for this is with the “Wait” parameter in your script. It’s simple enough to add; here is the batch file syntax:

@echo off
Start /Wait msiexec /i "applicationInstaller.msi" /qn

For PowerShell, you have two options. First, you can use the “-Wait” switch after Start-Process:

```
Start-Process -Wait msiexec "$($PSScriptRoot)\applicationInstaller.msi" -ArgumentList "/i /qn"
```

We can also pipe the output from msiexec to “Out-Null”, forcing it to wait for the application to complete.

```
msiexec /i "$($PSScriptRoot)\applicationInstaller.msi" /q | Out-Null
```

Assuming the app is packaged correctly using the [Intune Win32 Content Prep tool](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool), the installation should succeed and the ESP will display it as so.

Device only
-----------

The ESP has three phases: _Device Preparation, Device Setup, and Account Setup_. In most situations, I recommend skipping the account setup, or user phase. While I understand in some cases it may provide value, my experience generally tells us it’s unnecessary. This is especially true during a White-Glove deployment, where the device is provisioned based on apps and policies targeted to it at the device level, and the PC is ‘resealed’ before it is provisioned by the end user.

Disabling the account setup is simple. In the Microsoft Endpoint Manager, configure a custom policy and apply it to the “All devices” group. Here are the values for the policy:

-   **OMA-URI**: ./Vendor/MSFT/DMClient/Provider/MS DM Server/FirstSyncStatus/SkipUserStatusPage
    
-   **Data Type**: Boolean
    
-   **Value**: True
    

Finding a balance and final thoughts
------------------------------------

I often have a debate with Jesse, my solutions engineer, about how many apps we should force ESP to wait for. My rule is 8 or under, while his record is 23. And he’s not wrong. In theory, if the applications are configured correctly using the above methods, there should not be an issue.

Assuming everything is successful, we still want to think about provisioning time. Autopilot is about enabling end users to get up and running on a new PC quickly. There is no question that there are specific applications we want present on the device before the user reaches the desktop. But experience should also be considered. When provisioning passes the 20-25 minute mark, I believe it is a good opportunity to discuss which applications are critical vs what you’d be comfortable installing in the background while your users get to work.

Hopefully these tips will help you get through the ESP without issue. It can be incredibly frustrating to see failures here, so hang in there. Michael Niehaus has an incredible tool that can help show you what is happening during ESP in real-time, should you get stumped that can be found [here](https://www.powershellgallery.com/packages/Get-AutopilotESPStatus/2.0). As always, feel free to reach out with questions.
