---
author: GuestUser
date: Fri, 02 Oct 2020 20:34:44 +0000
description: '"Here’s a quick (but important) follow up on the first post. After some
  additional testing on our end, Steve Weiner and I were able to come up with a solution
  to assign applications to user-based groups, while excluding corporate (autopilot)
  devices at the same time.All Windows 10 devices"'
slug: personal-devices-and-the-intune-management-extension-part-2-another-psa-op82L
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/personal-devices-and-the-intune-management-extension-part-2-another-psa-op82L_thumbnail.jpg
title: Personal Devices and the Intune Management Extension Part 2 Another PSA
---

Here’s a quick (but important) follow up on the first post. After some additional testing on our end, Steve Weiner and I were able to come up with a solution to assign applications to user-based groups, while excluding corporate (autopilot) devices at the same time.

All Windows 10 devices have a basic registry path that acts as a placeholder for Autopilot settings. When devices are deployed through Autopilot, there are some additional registry settings that are populated with tenant-specific information. Here is one of the unique values that populates with Autopilot enrollment:

**Path:** Computer\\HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Provisioning\\Diagnostics\\AutoPilot

**Value**: CloudAssignedMdmId

The data for that particular value is the Azure tenant ID, but we don’t even need the data. For any intunewin packages that are assigned to user groups, we can add an additional requirement to install the app only if the registry value exists. See below:

![Picture1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1601670605027-TEXTWZS9IZXY42U02J6U/Picture1.png)

One more gotcha…

After we made these changes, we performed a test byo enrollment for the specific tenant. Things were looking good, as we saw the Intune Management Extension push down but none of the unwanted applications appeared to install. We even saw Intune reporting the application status as “Not applicable” with the required install. Perfect.

Then all of a sudden, Google Chrome installed. The surprise here was that Google Chrome was only assigned to an Autopilot dynamic device group, yet here it was on a personally enrolled device.

After some digging in Endpoint Manager and some logs, we noticed that the guid for Chrome was listed as a “ChildId” in the Intune Management Extension log:

![92181c91-a120-4353-a945-b44891bd1bab.jpg](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1601670698370-PT0VV26CLUF73C7TFUWY/92181c91-a120-4353-a945-b44891bd1bab.jpg)

What this tells us is that Google Chrome was listed as a dependency for the original application, and the console had the dependency configured to automatically install (For those who aren’t aware, the app guid appears in your browser’s url bar when looking at a specific application: [https://endpoint.microsoft.com/#blade/Microsoft\_Intune\_Apps/SettingsMenu/0/appId/11010011-abcd-efgh-1011-abcdefghijkl](https://endpoint.microsoft.com/#blade/Microsoft_Intune_Apps/SettingsMenu/0/appId/11010011-abcd-efgh-1011-abcdefghijkl)). In this screenshot, we are looking at the primary application to view the Chrome dependency:

![Picture1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1601670742747-0UVKTHYR2WNE1QKOFV8G/Picture1.png)

…so what does this mean? Intune will process and execute any required dependencies before it actually calculates the requirements for the primary application… yikes. 

But hey – there’s a solution. Add the same registry requirement to the chrome application so that it can’t install on any personal devices. I suppose you could also set the automatic install switch on the dependency to No, but I like the registry check better. And that’s it.

Anyway, hopefully that closes the gap on managing personal devices.
