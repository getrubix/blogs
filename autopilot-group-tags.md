---
author: steve@getrubix.com
categories:
- intune
- powershell
- azure
date: Sat, 30 Nov 2019 15:43:00 +0000
description: 'In the early days of Autopilot, we weren’t very concerned with the group tags (or ‘Order IDs). The real struggle was getting the hardware hash off the PCs and hoping they would register in Autopilot before hundreds of laptops arrived at a customer’s doorstep.'
slug: autopilot-group-tags
tags:
- intune
- script
- powershell
- azure
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-group-tags_thumbnail.jpg
title: Autopilot Group Tags
---

## The Beginning
---

In the early days of Autopilot, we weren’t very concerned with the group tags (_or ‘Order IDs_). The real struggle was getting the hardware hash off the PCs and hoping they would register in Autopilot before hundreds of laptops arrived at a customer’s doorstep. But now it’s not enough to just capture all of your computers in Autopilot; we need to assign varying deployment profiles to different groups of machines based on use case or type of hardware. I’ve created a PowerShell script that we can run manually on a device and choose the correct group tag.

![2019-11-30-14_37_11-test-no-ap-1903-on-sweiner-14178-virtual-machine-connection.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581088089392-N03YD1W4Q1HVNMC2Y9DT/2019-11-30-14_37_11-test-no-ap-1903-on-sweiner-14178-virtual-machine-connection.png)

There are plenty of write ups on the subject of dynamic groups based upon group tag as the membership rule. This is a particularly [good article](https://www.linkedin.com/pulse/building-windows-10-kiosk-without-self-deploying-mode-ewan-monro/) by Ewan Monro. Here’s a quick overview of what you need to do:

-   Create a dynamic device group in Azure AD with the membership rule (_device.devicePhysicalId_s _-any \_ -eq “\[OrderID\]:tagName”)_
    
-   Assuming your main Autopilot profile is assigned to a ‘catch all’ group _(device.devicePhysicalIds -any \_ -contains “\[ZTDId\]”)_, assign an alternative Autopilot deployment profile to the new dynamic device group.
    
## Think bigger
---

Now on a larger scale, the proper method for group tag assignment is at the procurement level. As long as your hardware vendor is properly setup for Autopilot enrollment, they should be able to assign group tags to your order. But for testing and existing devices, we need an easy way to choose a group tag when enrolling a device into Autopilot.

Before you start, make sure you read the [latest post on oofhours](https://oofhours.com/2019/11/29/app-based-authentication-with-intune/) by Michael Niehaus. This will give you the basics for setting up app-based Intune authentication with a PowerShell script. This is the first part of what I have done here:

![$clientId will be your application client ID that is generated when you register the app. $clientSecret will also be generated during that step; it’s all in Michael Niehaus’ write up.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581088172313-L1XYTJI643UOLCP556XQ/2019-11-30-15_13_53-e2978f-tagform.ps1-autoautopilot-visual-studio-code.png)

>`$clientId` _will be your application client ID that is generated when you register the app
<br>`$clientSecret` _will also be generated during that step; it’s all in Michael Niehaus’ write up._

This piece connects your script to Intune without username and password authentication. Next is the easy part; generating the hardware ID information. This will be the serial number and hardware hash. We’ll also make sure we append the group tag. Instead of the typical **Get-WindowsAutopilotInfo** script, we can just take the individual pieces and bake them into our script.

![2019-11-30-15_21_14-tagform.ps1-autoautopilot-visual-studio-code.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581088211423-51LDATXK2S8C5H8QX247/2019-11-30-15_21_14-tagform.ps1-autoautopilot-visual-studio-code.png)

But what about the group tag? First, look at the last part of the script, where everything gets assembled before being uploaded to Autopilot:

![2019-11-30-15_22_45-tagform.ps1-autoautopilot-visual-studio-code.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581088241108-F4DJTYN26CS1N0ASS2H1/2019-11-30-15_22_45-tagform.ps1-autoautopilot-visual-studio-code.png)

## Dynamic tag
---

Sure, we could just write a variable for the **$tag** and set it to whatever string we want. But we want something dynamic and efficient. We want to be able to choose from a selection of group tags every time we run the script. We can use a PowerShell list box form as seen in the screenshot at the beginning of this post. Here’s the code assembly for that piece.

![2019-11-30-15_25_05-tagform.ps1-autoautopilot-visual-studio-code.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090085416-MOD8IT157WJ1M6CGR46K/2019-11-30-15_25_05-tagform.ps1-autoautopilot-visual-studio-code.png)

That takes care of the box itself, but what about our options? We want to choose from a list of tags. I could’ve embedded this in the script, but I prefer to keep lists separate. That way, I can easily make changes without going back and editing the script itself. Here’s the XML list:

![Simple enough, right?](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090116179-D6VSGXYICORBR4QB8NQH/2019-11-30-15_27_45-config.xml-autoautopilot-visual-studio-code.png)

Simple enough, right?

The list will sit in my Azure blob storage for easy access. We can invoke a web request which will download the XML list and place it alongside my script. I’ll then parse out the options from the XML and use that for my list choices. Here is the code for that:

![2019-11-30-15_30_06-e2978f-tagform.ps1-autoautopilot-visual-studio-code.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090142803-1WGO1GJ19H7LVN0Y8587/2019-11-30-15_30_06-e2978f-tagform.ps1-autoautopilot-visual-studio-code.png)

You can see by the end, my **$tag** is taken from the selection that is made in the list. And that’s it!

For simplicity, I’ll usually place the PowerShell script alongside a _.bat_ file on a USB stick. This way instead of instructing someone to launch the PowerShell script with appropriate privileges, they just need to type something like _runMe.bat_– here is the contents of that:

`Powershell.exe -ExecutionPolicy Bypass -File .\groupTagScript.ps1 -Verb RunAs`

Once run, the device shows up under my enrolled devices in Intune. And of course, it has the proper group tag.

![2019-11-30-15_35_29-windows-autopilot-devices-microsoft-azure.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090170443-6MBW3IGJ4LRO1CS521QT/2019-11-30-15_35_29-windows-autopilot-devices-microsoft-azure.png)

While you will commonly see group tags utilized for Autopilot self-deploying kiosk devices, folks will now rely on them for devices like the new Surface Pro X, which can only run 32-bit applications. There has to be a clear way to separate the Pro X required apps and profiles from the rest of your standard fleet, which undoubtedly contains some 64-bit apps. This is a great way to achieve that.

Let me know if this method works out for you. Next, I’ll be working on gathering the actual group tags live from the Microsoft Graph.
