---
author: steve@getrubix.com
categories:
- intune
- powershell
- azure
date: Mon, 03 Feb 2020 20:09:00 +0000
description: "If you’ve been testing or deploying Windows 10 devices via Autopilot, it’s safe to assume you’ve encountered application failures during the device or user provisioning step of the ESP (enrollment status page). Now there’s nothing wrong with failures, but there is a problem when you can’t see which app is the troublemaker."
slug: exposing-esp-failures-with-the-microsoft-graph
tags:
- script
- endpoint manager
- azure
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/exposing-esp-failures-with-the-microsoft-graph_thumbnail.jpg
title: Exposing ESP failures with the Microsoft Graph
---

If you’ve been testing or deploying Windows 10 devices via Autopilot, it’s safe to assume you’ve encountered application failures during the device or user provisioning step of the ESP (enrollment status page). Now there’s nothing wrong with failures, but there is a problem when you can’t see which app is the troublemaker. On first glance, there’s no definitive log or event viewer that will provide insight as to what the problem is. Luckily, my team thrives on the elusive- we have a pretty solid (if not round-about) way to determine which apps are causing your ESP failures.

![2020-02-03-08_27_31-windows-enrollment-status-page_failure_thumb.jpg-672c397501.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105967161-IUZ7KAIWU5UIKRG75SYF/2020-02-03-08_27_31-windows-enrollment-status-page_failure_thumb.jpg-672c397501.png)

_\*Yes, of course if an app fails you will see that from the Microsoft Endpoint Manager_ _console, but often an app will end up installing just fine after a reattempt, while still leaving your ESP in chaos._

Finding the GUID
----------------

The first step when you see that wonderful failure screen is to get yourself to the event viewer. Navigate to **Applications and Services -> Microsoft -> Windows -> DeviceManagement-Enterprise-Diagnostics-Provider->Admin**. Filter by errors and warnings. Typically, application related failures are in the 1900-2000 Event ID range. Ah- seems like we found something:

![Which app is that?](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581106029630-SBVLSPTDPIZAQ37BW4GS/2020-02-03-08_31_16-batcave-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png)

Which app is that?

So there you have it! The app causing the issue is **CD035048-7A4D-4FF5-B52F-707A3221DAFD**. Must be a PDF editor- either way, that clears things up. Thanks for reading… what? Oh of course! That’s not the name of an application. That is just the Azure object ID of the app in GUID form. So now what?

Get to the Graph!
-----------------

Everything in Azure has a friendly name and an object ID. It just so happens that the MEM console does not feel like displaying the friendly name when it comes to applications. So if we had a way to connect that GUID with the application name, we would know which app is giving us the problem. This is exactly where the Microsoft Graph comes in.

If you’re not already familiar, the Microsoft Graph is essentially the spinal cord for all of Azure. It is a massive collection of all endpoints and APIs in raw data form. Luckily, the Graph Explorer is designed specifically for what we’re trying to do- search Azure for more information!

Go ahead and direct your browser to [https://developer.microsoft.com/en-us/graph/graph-explorer](https://developer.microsoft.com/en-us/graph/graph-explorer). Sign in with Global Administrator credentials.

![2020-02-07 15_05_23-Blog Feed – Inconvenient Integration.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581106057588-7M5N26PXP5AUKTRHRYXQ/2020-02-07+15_05_23-Blog+Feed+%E2%80%93+Inconvenient+Integration.png)

In the call URL, delete what is already there and type:

[**https://graph.microsoft.com/beta/deviceAppManagement/mobileApps**](https://graph.microsoft.com/beta/deviceAppManagement/mobileApps)

Click on the **Run Query** button. You may be asked to consent to the required permissions. If the query is successful, you will see a plethora of JSON text below.

![2020-02-03-08_25_20-graph-explorer-microsoft-graph.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581106080228-ET8R7O3H8VFJVJXPL6W2/2020-02-03-08_25_20-graph-explorer-microsoft-graph.png)

What’s with all this JSON?
--------------------------

Now that you’re staring at a giant notepad document filled with random garbage, you may be confused. This is the raw manifest of all the applications configured in Microsoft Endpoint Manager. They are displayed in the JSON format, which stands for _JavaScript Object Notation_. Essentially, this is just a list of objects and their properties. For this particular output, we should be able to see every property of an app- including it’s object ID GUID.

Go ahead and search for our mystery GUID on the page (Ctrl + F). Bingo:

![2020-02-03-08_32_37-_c__users_steve_desktop_apps.json-notepad.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581106137507-ULLSIGSAGV7UZD5A26YG/2020-02-03-08_32_37-_c__users_steve_desktop_apps.json-notepad.png)

Once you find your GUID, the display name should be just at the top of the object. In this case, we can see the app I was looking for ended up being my “AutopilotBranding” MSI, which is used to stage preferences during OOBE (if you haven’t checked it out yet, then I strongly recommend it: [https://github.com/mtniehaus/AutopilotBranding](https://github.com/mtniehaus/AutopilotBranding).

That’s it- hopefully this will help you figure out what’s going the next time ESP just tells you **Apps (Failed)**. Happy testing, and keep the questions coming!
