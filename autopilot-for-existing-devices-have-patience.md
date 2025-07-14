---
author: steve@getrubix.com
date: Mon, 24 Feb 2020 02:58:43 +0000
description: '"When a new PC is enrolled in Autopilot, it typically gets a deployment
  profile assigned within a 20 min time span.&nbsp; Assuming this is done by your
  hardware vendor, you don’t need to worry about those 20 min, because the clock starts
  when the device ships.&nbsp; Even if"'
slug: autopilot-for-existing-devices-have-patience
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-for-existing-devices-have-patience_thumbnail.jpg
title: Autopilot for existing devices have patience
---

When a new PC is enrolled in Autopilot, it typically gets a deployment profile assigned within a 20 min time span.  Assuming this is done by your hardware vendor, you don’t need to worry about those 20 min, because the clock starts when the device ships.  Even if you pony up for overnight shipping, that’s more than enough time for the computer to get a profile and have a terrific Autopilot experience.

But what about existing hardware that you are manually enrolling in Autopilot?  It’s critical that once you register the PC (either with a script or the CSV file) that you shut it down and do not attempt to go through the OOBE until it receives an “Assigned” status in the MEM portal.

![Screen Shot 2020-02-23 at 9.49.28 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1582512802112-9I7RW3IP3KBDORSOT0HN/Screen+Shot+2020-02-23+at+9.49.28+PM.png)

Understanding the process (the PC)
----------------------------------

When a Windows 10 PC boots up for the first, time, it goes through a specific that involves checking for Autopilot registration (assuming it is Windows 10 Pro version or higher).

-   **Network connection check**; the PC verifies that it has a network connection and can communicate with specific URLs.
    
-   **Authentication validation**; now that network connection is established and communication to Microsoft is open, Windows will check for an Azure AD device AUTH ticket.  This is essentially confirmation that the device is registered in Autopilot.  If the AUTH ticket is found, the device will proceed with Autopilot provisioning.  If not, it will proceed with the standard OOBE (Out of Box Experience).
    
-   When the AUTH ticket is found, the device receives the Autopilot provisioning information, including the JSON manifest that contains the Autopilot settings.  It will then proceed with the appropriate join and enrollment process.
    

Understanding the process (Azure)
---------------------------------

Up in the Modern Endpoint Manager (Intune) console, a device goes through several stages when it is registered with Autopilot.

-   Device is uploaded to Azure tenant, either by vendor or with the hardware hash CSV file.
    
-   Once uploaded, the device is dynamically added to a device group by group tag or physical hardware ID (Microsoft graph attributes).  Because the device is now in Azure, it is given an Azure Device ID object.
    
-   When the PC is in the correct group, an Autopilot profile is assigned.
    
-   It can take anywhere from 15-60 minutes for a profile to be assigned (although it is unusual for this process to take longer than 20 min).
    

What’s the problem?
-------------------

Again, for new devices there is no problem.  The registration happens before they’re shipped, so even with the fastest service on the planet, you’re not going to receive those devices before the profile has had enough time to assign.

For existing devices, I’d like to use a recent experience I had as an example.

I’ve been working with a customer that is trying to leverage Autopilot for existing machines in the field.  While these PCs are currently on Windows 7, the plan is to wipe them, update them to Windows 10 and then register them to Autopilot before enrolling in Intune.  Two weeks ago, in testing, we noticed something unusual.  Our customer was stating that these devices were booting into the standard user experience, with no sign of the custom OOBE that Autopilot provides.  Furthermore, once on the desktop, there were no signs of any Intune enrollment, policy or applications.  Here is what we saw:

![SC1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1582512937546-V1PTTGIERZYJQ5JCNISJ/SC1.png)

When we check the console, it’s clear the device is not only registered, but has a profile assigned to it.  Theoretically, this means the device should be enrolled in Intune.  But it’s absolutely not.  We know this not just because there are no profiles pushing to the device but take a look at the **Enrollment state** and the **Associated Intune device** attributes; not enrolled and no associated Intune device.

How could this be?
------------------

The mystery here is how a device that is registered in Autopilot with an assigned profile could sneak by an Intune enrollment?  It has us scratching our heads for a bit, until we thought about the particular process this customer was using.  They were updating to Windows 10, registering the device to Autopilot, and enrolling in Intune…with no time in between!  Don’t forget the last step of Autopilot registration as mentioned above:

_\*It can take anywhere from 15-60 minutes for a profile to be assigned_

Ah ha!  It was now clear.  If you try to proceed with the OOBE before the device has been assigned a profile, then it makes complete sense it would find no AUTH ticket when attempting to reaching out to Autopilot (which is what we ultimately saw in the event viewer logs).  

That was it?
------------

You can imagine that the subsequent testing was fairly simple- we reset the PC and started over, making sure the profile was assigned before moving forward.  And of course, we ended up with a successful enrollment.

I wanted to write about this one because while it wasn’t an amazing technical fix, it goes to show how nuanced Autopilot can be, especially given how new it is.  And when you move away from brand new, out-of-the-box devices, things get even wonkier. 

I plan on detailing in a future post exactly how you can go from Windows 7 to 10 and enroll in Autopilot at the same time.  But beware; it is not for the faint of heart.
