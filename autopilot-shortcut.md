---
author: steve@getrubix.com
categories:
- intune
- powershell
- azure
date: Wed, 04 Mar 2020 16:13:09 +0000
description: >
  I’ve been working on a write up about co-management and the SCCM cloud management
  gateway for a week or so now, and it’s becoming about as painful as the actual
  process itself. So in the meantime, I wanted to share a handy little trick I came up with.
slug: autopilot-shortcut
tags:
- intune
- script
- powershell
- azure
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-shortcut_thumbnail.jpg
title: Autopilot Shortcut
---

I’ve been working on a write up about co-management and the SCCM cloud management gateway for a week or so now, and it’s becoming about as painful as the actual process itself.  So in the meantime, I wanted to share a handy little trick I came up with for quickly adding devices to Autopilot for testing or other purposes.  

By now, you’re probably familiar with the standard PowerShell script process which is as follows:

```
Set-ExecutionPolicy Bypass -Scope Process -Force
```

```
Install-PackageProvider -Name NuGet -Confirm:$false -Force
```

```
Install-Script -Name Get-WindowsAutopilotInfo -Confirm:$false -Force
```

```
Get-WindowsAutopilotInfo.ps1 -OutputFile “C:\pathToHardwareHash.csv”
```

But that’s just the process for generating the hardware hash file.  You then have to log into the Azure portal and upload that file manually.  If you’re like me and have done this too many times to count, then you’ve probably thought “there has to be a better way!” (Yes, I should have done infomercials).  

So let’s dive into how you can add your device to Autopilot from PowerShell with just one line.

Body of the script
------------------

From a high level, we’re going to create everything we need for generating the hardware hash and posting it to Autopilot with the Microsoft Graph in one shot.  Let’s a have a look:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1583337959758-REDQJFKX0MXTBNN99DOC/image-asset.png)

You’ll notice for the authentication piece, I went with an interactive logon which will give us a multi-tenant solution:

```
Connect-MSGraph -ForceInteractive
```

This is because I have a hand in about 30 Azure tenants at one time, and need to choose which tenant to send a device to every time I run the script.  If you’d prefer to run this without logging in every time, check out Michael Niehaus’s guide on [app-based authentication for Intune](https://oofhours.com/2019/11/29/app-based-authentication-with-intune/).

Keep it in a blob
-----------------

The problem with scripts like this is you need to put them on a USB drive to run them on new devices.  And if you’re pulling a .csv file from the machine, you’ll need that drive anyway.  But this is 2020- we shouldn’t need drives.  We’re going to store our script in an Azure Storage Blob so we can call it ourselves from PowerShell and run it in the same line.

If you’re new to Azure Blob Storage, follow this guide to get going: [https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-portal](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-portal).  I also highly recommend using the Azure Storage Explorer desktop app- it makes managing your files in the blob a breeze.

Take the above script and place it in a storage container.  Note the URL path to the file.  Keep in mind that we will be typing this each time to run it, so don’t make the name unbearable.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1583338033080-GPSMZ70LBOWVUCW09N1B/image-asset.png)

Register with one line
----------------------

On the device you want to register in Autopilot, use Fn+Shift+F10 (assuming you’re at the OOBE) and type powershell.  Press enter.

Use the following line to invoke and run the script:

```
Set-ExecutionPolicy Bypass -Scope Process -Force; Invoke-Webrequest ‘https://urlPathToStorage.blob.core.windows.net/container/script.ps1' -OutFile C:\script.ps1; C:\script.ps1
```

Your script will be pulled from blob storage, run on the machine, and finally delete itself.  You will see the graph post results displayed after a successful Autopilot registration:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1583338114754-PU7ZWZBIT4N8Z0MI4T9Q/image-asset.png)

Enjoy, and let me know your thoughts!
