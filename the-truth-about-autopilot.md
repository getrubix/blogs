---
author: steve@getrubix.com
categories:
- intune
- security
- automation
- azure
date: Wed, 17 Jul 2019 22:42:00 +0000
description: "Windows Autopilot deployment services enable new, Windows 10 PCs to be ‘business ready’ out of the box so your employees can get to work faster. At least that’s what the internet would have you think. Over the past two years since it’s official release, I’ve helped organizations all"
slug: the-truth-about-autopilot
tags:
- automate
- azure
- active directory
- security
- intune
- flow
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/the-truth-about-autopilot_thumbnail.jpg
title: The truth about Autopilot
---

Windows Autopilot deployment services enable new, Windows 10 PCs to be ‘business ready’ out of the box so your employees can get to work faster. At least that’s what the internet would have you think. Over the past two years since it’s official release, I’ve helped organizations all over the world deploy Windows 10 with Autopilot. And the reality is, Autopilot does very little.

That’s not a knock against Autopilot; it functions exactly how it’s supposed to, which is to act as an automated enrollment tool for Windows 10 PCs. If you’ve had any experience with Apple’s Device Enrollment Program (DEP), then you already understand Autopilot. Here is the flow:

-   Windows 10 PC is purchase from OEM, vendor or reseller and registered in Autopilot
    
-   The device is then tied to your Azure Active Directory tenant. It should be visible in your Azure portal before the device even ships from it’s origin
    
-   When the PC boots up for the first time, it calls out to Azure and checks if it belongs anywhere; if registered in Autopilot, users will be brought to a custom OOBE (Out of box experience) in which they’re prompted for their corporate Active Directory credentials.
    
-   Assuming credentials are valid, the PC is Azure AD joined and passed off to the automatic Intune MDM enrollment
    

![image_thumb373-e1563731747452.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581032505797-8TNCR8CQKC4XCMGCRAEL/image_thumb373-e1563731747452.png)

If that all goes to plan, you now have a device that’s enrolled in Intune; Microsoft’s endpoint management tool. But what next? When do the device restrictions, update policies, applications and the rest of the good stuff push to the device? The answer is it won’t unless you’ve built them out in Intune. And that is the common misconception. Autopilot will enroll you with Intune faster and with less chance of errors, but you need to have an infrastructure built out to manage the device. For all of this to work, you need:

-   An EMS license (Enterprise Mobility + Security) for each user enrolling
    
-   New hardware shipping with Windows 10 build 1709 or later (1809 is preferred)
    
-   A procurement source that has the capability to auto enroll PCs for you
    
-   Someone to put this all together for your organization
    

The benefit of using Autopilot to deploy Windows 10 hardware is the ability to drop ship a device to your users regardless of their location and without worrying about them being on or off the domain. Autopilot and Intune authenticate over Azure AD, so folks can literally do this at a Dunkin’ Donuts.

Usually at this point, there are quite a few questions to be answered such as:

-   “If the PC doesn’t join the domain, how does it get group policy?”
    
-   “When do I image the PC for employee use?”
    
-   “How do I push the SCCM (System Center Configuration Manager) client to the PC?”
    
-   “WHY AREN’T I JOINING THESE TO MY DOMAIN?”
    

Those who’ve been doing their homework know these answers and are ready to turn the key on modern management. For the rest, stay tuned; I’ve got a lot more coming.
