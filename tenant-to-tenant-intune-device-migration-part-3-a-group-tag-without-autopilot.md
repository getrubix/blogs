---
author: steve@getrubix.com
date: Mon, 31 Jul 2023 18:58:40 +0000
description: '"Today was supposed to be covering the GroupTag and SetPrimaryUser tasks.
  But after I started digging into this one, it became clear that it needed it’s own
  post. So let’s jump in.If you’re just joining us, start here. GROUP TAG (TRIGGER:
  5 MIN AFTER LOGON)Group tags"'
slug: tenant-to-tenant-intune-device-migration-part-3-a-group-tag-without-autopilot
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/tenant-to-tenant-intune-device-migration-part-3-a-group-tag-without-autopilot_thumbnail.jpg
title: Tenant to Tenant Intune Device Migration Part 3 A Group Tag Without Autopilot
---

Today was supposed to be covering the **GroupTag** and **SetPrimaryUser** tasks. But after I started digging into this one, it became clear that it needed it’s own post. So let’s jump in.

_If you’re just joining us, start_ [**_here_**](https://www.getrubix.com/blog/tenant-to-tenant-intune-device-migration-part-1)_._

### **GROUP TAG (TRIGGER: 5 MIN AFTER LOGON)**

Group tags (also known as _DevicePhysicalIds)_ are Azure AD device attributes that are typically used for Autopilot registration. They allow the device to be dynamically grouped based on this attribute before it is enrolled to Intune and we use them throughout the lifecycle of the device.

_Last year, I wrote a_ [_5-part deep dive on the group tag reference architecture_](https://www.getrubix.com/blog/autopilot-group-tags-1)_, so be sure to check it out if you haven’t._

It is critical that when the PC moves from Tenant A to Tenant B, it retains the same group tag to ensure proper assignment of Intune profiles. But wait- why aren’t we just applying the group tag when we register this device to Autopilot in an upcoming script? And more curious, how do you apply a group tag _without_ Autopilot?

We’re not running the **AutopilotRegistration** task until later, to ensure the device record has had time to be removed from Tenant A. But we can’t wait until then for the tag because we need the device in the right group as soon as possible. Now for the best part; adding a group tag without Autopilot registration.

Let’s take a look in the Microsoft graph API explorer and navigate to the **devices** endpoint. Here is what a standard, non-Autopilot, device looks like when joined to Azure AD without a group tag:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7662e7bc-a3a7-4515-bb84-07f35b700b72/noTag.jpg)

Notice the “PhysicalIds” attribute section.

Now let’s take a look at an Autopilot registered device with a group tag:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/f5b207c2-782a-4300-bc44-5fc0b7116b30/withTag.jpg)

There are two additional entries in the PhysicalIds attribute section; **ZTDID** and **OrderID**. What do these mean? The **ZTDID** is the Autopilot registration entry. The **OrderID** is our group tag, which makes sense if you have written any dynamic Azure AD group rules for devices before. So knowing what the group tag attribute is and where it needs to go, we just need to add it directly to the graph.

Our PowerShell script for this contains three important pieces of information:

1.  The group tag name from Tenant A
    
2.  The **OrderID** attribute with our tag in JSON format
    
3.  The patch call to the Azure device endpoint
    

As always, we start by authenticating to the graph with an app registration, this time to Tenant B. Then we need to grab the group tag from the device in Tenant A… uh oh! Didn’t we remove the device from Tenant A Autopilot? We sure did, but don’t panic- remember, we stored some key info in our local **MEM\_Settings.xml** file. The group tag from Tenant A can be retrieved from the XML and stored in a variable.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/8d6a121e-feb1-4163-99a3-cc37e6d3c599/script1.jpg)

Next we have to get the Azure AD device ID in Tenant B. Problem; all we have to look the device up is the serial number, and Azure AD doesn’t store that as an attribute. But Intune does. And Intune objects are associated with Azure AD objects. Time for some correlation:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/0a659936-2dc4-467e-ac57-6a38ec8220d6/Screenshot+2023-07-31+at+2.43.06+PM.png)

By looking up the serial number in Intune managed devices, we can retrieve the Azure AD object and store the ID in a variable. We know where the tag is going, now we just need to format it.

The group tag can not be added by itself. First we have to get the entire **PhysicalIDs** entry and append our tag to that.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/24373b9c-f73c-45ac-84a3-db6bcfbe826b/Screenshot+2023-07-31+at+2.45.36+PM.png)

Our **$oldTag** variable is placed inside of a string with **\[OrderID\]**, so it will blend in. After the tag is added to the IDs, we can convert it to JSON:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/2878693e-0e8a-4be7-ac5c-6cafec1933ba/Screenshot+2023-07-31+at+2.47.29+PM.png)

Easy! Now all that’s left is to invoke the PATCH call to the Azure AD object:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/117dd0de-0061-49ba-8c90-e329bb82c64c/Screenshot+2023-07-31+at+2.48.23+PM.png)

The patch is fairly quick, so the device should be placed in the right dynamic group shortly after. And that’s it; we successfully added a group tag attribute to a device without Autopilot registration.

Next time…
----------

Both the **GroupTag.ps1** and **GroupTag.xml** have been uploaded to the [Intune Migration repo](https://github.com/groovemaster17/IntuneMigration/tree/main). I promise, next post will cover the **SetPrimaryUser** task.
