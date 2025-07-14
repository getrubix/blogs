---
author: steve@getrubix.com
date: Sat, 25 Jan 2020 20:04:00 +0000
description: '"Here we go! This one has been a long time coming, so thanks for waiting
  around. Most folks know that until recently I have been extremely against hybrid-joining
  a PC with Autopilot. It was meant for pure, Azure AD join only, and damnit, that’s
  the way it should"'
slug: autopilot-for-hybrid-join-a-somewhat-visual-guide
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-for-hybrid-join-a-somewhat-visual-guide_thumbnail.jpg
title: Autopilot for Hybrid join a somewhat visual guide
---

Here we go! This one has been a long time coming, so thanks for waiting around. Most folks know that until recently I have been extremely against hybrid-joining a PC with Autopilot. It was meant for pure, Azure AD join only, and damnit, that’s the way it should stay. But as the technology itself matured, and industries of all sizes are heading towards the modern desktop, I (and my reluctant team) must adapt.

As I put the final touches on this post, I’m looking at another 9 that are already in the works. This is a heavy topic and has a lot of components. First and foremost, what is Autopilot hybrid join and how does it work?

_Spoiler alert: this is where the misconceptions get broken down._

-   Hybrid join (or Hybrid Azure AD join) is the act of domain joining a PC and letting it register to Azure AD via Azure AD connect. The machine is **NOT joined to both Azure and the domain**. As I’ve said before- join once and register once.
    
-   Autopilot can facilitate Hybrid join **without an admin** needing to log in first to join the PC to the domain (more specifically, the Intune connector does this, but we will get to that later).
    
-   The machine being enrolled **MUST BE PHYSICALLY ON THE DOMAIN**. I cannot stress that part enough. VPN does not work, nor do PowerShell scripts or other hacks. This is non-negotiable.
    
-   There is still no technical requirement to actually do this. **Modern desktop is designed for Azure AD join** machines fully managed by Intune, leaving both the domain and group policy in the dust. It is inevitable that everyone will get there, but the question is simply how long. So this is a good stepping stone.
    

Alrighty- ready to go? Have a look at the high level reference architecture for Autopilot Hybrid join.

![It’s almost like a theme park](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105584573-3W5DQP8195DRUYOC5W90/2020-01-25-15_31_05-refarch_creator.pptx-powerpoint.png)

It’s almost like a theme park

Still with me? Good, because it’s not as scary as it seems. I’m not going to go through all the hard requirements, because that bleeds into what you need for standard Autopilot, Windows 10 management, licensing, blah blah, etc. If you need those, Google is free or just click [here](https://docs.microsoft.com/en-us/intune/enrollment/windows-autopilot-hybrid).

Let’s take a look at what happens to a device as it goes through the Autopilot Hybrid join process. I’ve broken this down into parts, each with it’s own steps.

Part 1: Registration and searching
----------------------------------

![2020-01-25-15_53_24-refarch_creator.pptx-powerpoint.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105618524-L8N665AGYMN4SPPCQ89I/2020-01-25-15_53_24-refarch_creator.pptx-powerpoint.png)

1.  Windows 10 PC is registered to Autopilot, via [PowerShell script](https://www.powershellgallery.com/packages/Get-WindowsAutoPilotInfo/1.6) or by your hardware vendor.
    
2.  PC receives an Autopilot deployment profile specifying it will be Hybrid joined.
    
3.  Autopilot communicates this to Intune, which then checks if a domain join configuration profile exists.
    
4.  This whole time, the PC is just constantly polling for a domain controller, which it will not find as it does not have the proper configuration. It only knows it needs to find one because of the Hybrid join Autopilot profile
    

Part 2: Requesting a blob
-------------------------

![2020-01-25-15_04_47-refarch_creator.pptx-powerpoint.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105665057-5TX4HVW970KEINSAH59M/2020-01-25-15_04_47-refarch_creator.pptx-powerpoint.png)

1.  Intune locates the Domain join configuration profile. This contains the PC name prefix to be applied, the domain controller name, and the specific organizational unit (OU) that the PC will belong to once it joins.
    
2.  Having the configuration profile, Intune makes a request to the Intune connector (on the local domain) for an ODJ (_Offline Domain Join_) blob\*. This is basically a manifest that the PC will use to join the domain.
    
3.  After verifying the request with the local domain controller, the Intune connector sends the ODJ blob back to Intune to be sent to the PC.
    

\*_While it is called an ‘Offline Domain Join’ blob, the PC must have line-of-sight to the domain controller_

Part 3: Meet and reboot
-----------------------

![2020-01-25-15_08_41-refarch_creator.pptx-powerpoint.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105688029-7UISAT2Y5B5DLOB8K1A3/2020-01-25-15_08_41-refarch_creator.pptx-powerpoint.png)

1.  Intune pushes the ODJ blob to the PC, which now knows the domain controller it needs to look for.
    
2.  The PC finally sees the domain controller (again, line of sight is required- NO VPNs).
    
3.  PC reboots to begin domain join.
    

Part 4: The joining
-------------------

![2020-01-25-15_20_41-refarch_creator.pptx-powerpoint.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105715782-WTLBU0SM1EIQQ70AVAVO/2020-01-25-15_20_41-refarch_creator.pptx-powerpoint.png)

1.  The domain controller uses delegated computer permissions to place the PC in the specified OU.
    
2.  PC is now domain joined.
    
3.  Once domain joined, the PC can receive Group Policy objects and certificates, just as a traditionally joined machine would.
    

Part 5: The circle is complete
------------------------------

![2020-01-25-15_30_19-refarch_creator.pptx-powerpoint.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105759199-SKX862HCSCAHGI69SKQQ/2020-01-25-15_30_19-refarch_creator.pptx-powerpoint.png)

1.  Azure AD connect will attempt to register the PC during the next sync.
    
2.  The PC is successfully registered to Azure AD (technically _re-registered_ but more on that in a future post).
    
3.  Now belonging to Azure AD and the local domain, the PC can receive applications and profiles from Intune in addition to Group Policy and other local resources.
    

And there you have it! As I stated earlier, there are a lot of components to this, each of which will probably end up being it’s own post. And with more components, that means more fail points; yay! But I am hoping this will at least clear up some things if you’re considering a Hybrid join deployment with Autopilot.

Ya know, after reading this whole thing with the illustrations, the ‘father of twins’ part of me thinks it would make a good children’s book. Better see what my wife thinks about it.
