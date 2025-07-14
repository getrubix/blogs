---
author: steve@getrubix.com
categories:
- intune
- powershell
date: Mon, 14 Mar 2022 17:54:16 +0000
description: '"While working on the next Group Tag installment, I realized I left
  out two interesting concepts that I should probably address before moving forward.
  How does a Group Tag structure impact device naming, and where does user assignment
  fit into all this mess?What’s in a name?Naming devices"'
slug: autopilot-group-tags-part-3
tags:
- intune
- endpoint manager
- powershell
- script
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-group-tags-part-3_thumbnail.jpg
title: Autopilot Group Tags Part 3
---

While working on the next Group Tag installment, I realized I left out two interesting concepts that I should probably address before moving forward. How does a Group Tag structure impact device naming, and where does user assignment fit into all this mess?

What’s in a name?
-----------------

Naming devices with Autopilot is a fairly simple process (unless you are choosing to hybrid join your devices, in which case naming is the least of your issues). In the deployment profile, you can choose a combination of static and dynamic values to create unique names. Most organizations tend to use something like this:

**ABC-%SERIAL%**

Don’t worry if the serial and prefix are longer than 15 characters (the Windows name limit)- the serial will get truncated automatically. But what does this have to do with group tags? We just reviewed that the name is set in the Autopilot deployment profile. And where are we assigning these? Exactly; the dynamic groups based on our tags!

Let’s reference the Group Tag structure we made for Magic Coffee Co.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/2c2298fb-c52f-418d-9605-367b8f456834/All.png)

They want to name their devices based on the site location. Their PC naming schema looks like this:

**MC-<SITE>-<5 RANDOM NUMBERS>**

So, MC (for Magic Coffee), the site abbreviation and some random numbers to fill out the rest. Here are some possible names:

-   MC-NY-98556
    
-   MC-LA-34411
    
-   MC-CH-02216
    

Okay, you get the picture. So now all we have to do is create three, different Autopilot profiles, and have them named accordingly. This way, devices will be automatically named based on their Group Tag.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/0d32f885-c87a-4491-b559-ec04daf0bc00/Screen+Shot+2022-03-14+at+11.38.27+AM.jpg)

There are alternative ways to name devices using Intune that involve cleverly timed, PowerShell scripts. But unless you absolutely need to, I would say it’s not worth the hassle.

Device names are becoming less important than they used to be given the visibility we have in Endpoint Manager to _who_ is using _what_. Next up, I’ll touch on the user assignment part of this whole thing.
