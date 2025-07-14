---
author: steve@getrubix.com
date: Fri, 24 Jul 2020 18:20:28 +0000
description: '"Welcome to Part 2 of deploying apps with package managers.&nbsp; Today
  is all about Winget, the Microsoft package manager.&nbsp; A few things to know before
  we jump in:Winget will be available in Windows 10 2004 in an upcoming release. For
  now, use insider build 10.0.20150.1000.If you don''t"'
slug: intune-and-package-managers-part-2-winget
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/intune-and-package-managers-part-2-winget_thumbnail.jpg
title: Intune and Package Managers - Part 2 Winget
---

Welcome to Part 2 of deploying apps with package managers.  Today is all about Winget, the Microsoft package manager.  A few things to know before we jump in:

-   Winget will be available in Windows 10 2004 in an upcoming release. For now, use insider build **10.0.20150.1000**.
    
-   If you don't feel like making the jump to 2004, you can simply deploy the updated desktop installer from [https://github.com/microsoft/winget-cli/releases](https://github.com/microsoft/winget-cli/releases) (this is the method we'll be using today).
    
-   General information on Winget can be found [here](https://github.com/microsoft/winget-cli).
    

But why use Winget instead of Chocolatey (which was covered in [Part 1](https://www.getrubix.com/blog/intune-and-package-managers-part-1-chocolatey))?  Well, like most answers related to Microsoft, it depends.  Chocolatey has no doubt been around longer and therefore a tad more robust in terms of its capabilities such as updates, uninstalls, etc.  It seems that the biggest feedback I receive from folks when choosing a package manager is that the Choco repository is very "open" (unless you want to build your own private repo).  Granted, most packages on Chocolatey are verified by the open source community and validated frequently, you can imagine most security departments taking issue with this.

In fact, every time I would suggest Chocolatey as a way to streamline app deployment, people would always follow up with "too bad Microsoft doesn't have their own version".  Well, here we are.  And I'll admit, there is something to be said for leveraging a platform with Microsoft's branding on it as opposed to others.  So let's get to packaging!

Meet the Microsoft Package Manager
----------------------------------

The schema for installing Winget packages is fairly straight forward:

```
winget install --exact --silent "ApplicationId"
```

The “Id” variable comes from the way Winget lists apps.  Simply type **winget search** to view all available packages, and you'll notice the three columns; Name, Id, and Version. 

![2020-07-24 13_01_39-Administrator_ PowerShellV20.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595613645641-OP66LFOPNHC44SORVMG0/2020-07-24+13_01_39-Administrator_+PowerShellV20.png)

If you were to simply use the "Name" parameter, then Winget may throw something that looks like this:

![Untitled picture.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595613972587-CXIJKIVLBXMC9F1WE65X/Untitled+picture.png)

Looks like our command will have to be a bit more specific.  That's where the Id parameter comes in handy, coupled with the "--exact" switch.

![2020-07-24 13_07_44-ZTDS-TRAINING - VMware Workstation.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595614132401-YXWMR0BUVRZTTLGQ8HI2/2020-07-24+13_07_44-ZTDS-TRAINING+-+VMware+Workstation.png)

There we go!

All we have to do is create an install script, package it, and then push it the same way we did with Chocolatey, right?  Well, no.  Remember- Winget isn't included in Windows yet by default, so we're gonna have to push it to the device first, either stand alone or as a dependency. 

Pushing Winget
--------------

We don't technically push Winget itself, but instead the updated Windows Desktop App Installer application found [here](https://github.com/microsoft/winget-cli/releases).

Log into [https://endpoint.microsoft.com](https://endpoint.microsoft.com) and head to **Apps -> Windows -> Add+**.  Select **Line-of-business app** from the drop down.

![2020-07-24 10_30_01-Select app type - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595614255217-BXEGAP7WWN2MM39NENBI/2020-07-24+10_30_01-Select+app+type+-+Microsoft+Endpoint+Manager+admin+center.png)

Upload the appxbundle for the Desktop App Installer.  If you have the right file, the Endpoint Manager will now complain to you that it needs dependencies to push the app. 

![2020-07-24 10_31_20-App package file - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595614304507-ICGAFBWYRYKAO7S5LYT8/2020-07-24+10_31_20-App+package+file+-+Microsoft+Endpoint+Manager+admin+center.png)

You can find everything you need right here: [https://store.rg-adguard.net/](https://store.rg-adguard.net/).  Change the search parameter to **PackageFamilyName**, and search "Microsoft.DesktopAppInstaller\_8wekyb3d8bbwe".  All of the dependencies can be downloaded here.

![2020-07-24 13_20_40-Microsoft Store - Generation Project (v1.2) [by @rgadguard & mkuba50] and 1 more.png](https://images.squarespace-cdn.com/content/v1/5dd365a31aa1fd743bc30b8e/1595614340906-6S45TBOXE6KHQCNNFEBZ/2020-07-24+13_20_40-Microsoft+Store+-+Generation+Project+%28v1.2%29+%5Bby+%40rgadguard+%26+mkuba50%5D+and+1+more.png)

Be sure to only download the .APPX files.  Upload to Endpoint Manager and everything should check out, allowing you to assign the app to all devices.

![2020-07-24 13_25_29-App package file - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595614422729-M8ZL86253PYFENX94U9U/2020-07-24+13_25_29-App+package+file+-+Microsoft+Endpoint+Manager+admin+center.png)

Because we need the Updated Desktop Installer on the device before we start pushing apps, go ahead and install it during Autopilot provisioning, or set it as a dependency for the individual apps.

Wrap it up
----------

For this, we're going to deploy VLC media player.  Create an install directory, and place a PowerShell script inside called **InstallVLC.ps1**

![2020-07-24 13_28_27-● InstallVLC.ps1 - Visual Studio Code.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595614491151-GIUFV99L57TWC5ND8VHV/2020-07-24+13_28_27-%E2%97%8F+InstallVLC.ps1+-+Visual+Studio+Code.png)

Go ahead and use the [Microsoft Win32 Content-Prep-Tool](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool) to create an .INTUNEWIN file from the script.  Upload to the Endpoint Manager and configure with the following parameters:

**Install command:** powershell.exe -executionpolicy bypass -windowstyle hidden .\\InstallVLC.ps1

**Uninstall command:** "C:\\Program Files\\VideoLAN\\VLC\\uninstall.exe"

**Install behavior:** User**\***

**\***_This part is critical.  Winget uses the Desktop App Installer to deploy applications, and that ONLY runs at the user level.  This MUST be set to "User" to work- setting to "System" would result in the Intune Management Extension not being able to find the specified path of the Desktop App Installer._

![2020-07-24 13_36_49-Edit application - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595614620314-AW9FO5VW0RX4OYM1FKXJ/2020-07-24+13_36_49-Edit+application+-+Microsoft+Endpoint+Manager+admin+center.png)

The detection rule is simple.  I'm using the folder path of **C:\\Program Files\\VideoLAN\\VLC** and checking if it exists.

![2020-07-24 13_41_58-Detection rule - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595614653797-QLWQBTA4U3WQKNGMN188/2020-07-24+13_41_58-Detection+rule+-+Microsoft+Endpoint+Manager+admin+center.png)

I'm assigning the application as available so users can access it from the Company Portal.  It installs with no issue.

![2020-07-24 13_50_57-ZTD-AP-EUR-1909 - VMware Workstation.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595614679322-T2KWM33E7T5THDH085KJ/2020-07-24+13_50_57-ZTD-AP-EUR-1909+-+VMware+Workstation.png)

So which will you use- Chocolatey or Winget?  As I've stated in the beginning of Part 1, there are pros and cons to each, and for that matter, pros and cons of using a package manager at all.  But for me, anything to make things more streamlined is a good thing.  And isn't that the whole point of "Zero Touch"?
