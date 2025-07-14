---
author: steve@getrubix.com
date: Tue, 15 Mar 2022 12:37:23 +0000
description: '"Autopilot Group Tags can play a major role in application deployment.
  Devices are registered during procurement with a tag applied, the tag ensures the
  device ends up in the appropriate group, and that group is used to assign applications
  to be deployed during initial provisioning. But"'
slug: autopilot-group-tags-part-4
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-group-tags-part-4_thumbnail.jpg
title: Autopilot Group Tags Part 4
---

Autopilot Group Tags can play a major role in application deployment. Devices are registered during procurement with a tag applied, the tag ensures the device ends up in the appropriate group, and that group is used to assign applications to be deployed during initial provisioning. But what about user assigned apps? How do they play into a successful deployment? Let’s have a look.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/587775e0-7adf-41b4-b362-a6242c6907ea/Screen+Shot+2022-03-14+at+4.49.43+PM.png)

A typical Autopilot onboarding flow

A balancing act
---------------

I’ve always seen Autopilot provision as a balancing act. We want to make sure all required applications and profiles are provisioned during that first enrollment so the user has everything they need to get to work. On the other hand, the whole point of Autopilot is to provide a efficient user experience. Therefore, we cannot accept a user waiting for hours while the PC sets itself up.

If you’re not already familiar with the Enrollment Status Page (ESP), I recommend checking out my first two posts about it:

-   https://www.getrubix.com/blog/please-wait
    
-   https://www.getrubix.com/blog/autopilot-white-glove-what-happens-between-hitting-the-windows-key-5-times-and-getting-to-a-hopefully-green-screen
    

Understanding and mastering the way ESP works is crucial for a successful Autopilot deployment. Let’s think of the provisioning process in two pieces: device setup (during ESP) and user setup (post ESP). Device setup should closely resemble anything that would be part of a traditional image. This includes core applications, policy and settings, and OS customizations. This works especially well with Autopilot pre-provisioning (formerly known as “White Glove”) when you want to front-load all of the device targeted applications.

Each step
---------

Let’s go ahead and break down the provisioning process during an Autopilot deployment. I’m going to address each part as a step in a phase:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/40e5c76b-d47d-45f6-a46f-da3b1e299c55/Screen+Shot+2022-03-14+at+4.44.14+PM.png)

We have two phases and three steps. The **device level** phase consists of two parts: the initial sign-in and the device provisioning. The **user level** phase has just one step, which is the actual user log on.

Device phase
------------

### A. OOBE

View fullsize

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/8bcc0e0b-64bc-48a3-92c0-1de624b1408d/Screen+Shot+2022-03-14+at+8.54.04+PM.png)

The first part of the device phase takes place when the end user powers on the Autopilot device and goes through the OOBE (Out-of-Box-Experience). This is where the user is greeted with the company branding and prompted to enter their corporate credentials. Most folks think this is where the user logs into the PC, but that’s not the case.

So what is actually happening here? The credentials that the user entered are authenticated by Azure AD and Intune to verify the following:

-   Does the user have an Azure AD Premium and Intune license assigned?
    
-   Is the user authorized to Azure AD join a device?
    
-   Is the user authorized to enroll to Intune?
    
-   Does the Azure AD device object match the associated Autopilot object?
    

Once these components are validate, the device proceeds to the next step where it begins Intune enrollment. Keep in mind, there is no user object present on the device at this point.

* * *

### B. Device provisioning

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/a38d079f-3262-4544-b77f-39a4dab5fe07/Screen+Shot+2022-03-14+at+9.00.18+PM.png)

This where the core provisioning happens in the device setup. All apps and policies that have been assigned to the device based on its group tag are now deployed. Once provisioning is finished, the **device level** phase is now complete. If this device were being pre-provisioned, then this is where the process would complete.

> I always recommend skipping the user/account tracking portion of the ESP. For information on why and how, read [here](https://www.getrubix.com/blog/autopilot-white-glove-what-happens-between-hitting-the-windows-key-5-times-and-getting-to-a-hopefully-green-screen).

Once device setup is complete, we move onto the next phase. There is still no user object on the device.

* * *

User phase
----------

### C. User setup

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e23c9228-8814-4faf-ad9c-2e994b5a5392/Screen+Shot+2022-03-14+at+9.03.00+PM.png)

After device provisioning, the user is prompted with an official, Windows logon screen. This time when they enter their credentials, they are actually logged in. The user object is now present, the user profile is cached, and the device is issued a [Primary Refresh Token](https://docs.microsoft.com/en-us/azure/active-directory/devices/concept-primary-refresh-token) to associate it with the user.

It is at this point that any user assigned applications can be deployed. If you’ve set your assignments to _required_, then they will simply download and install in the background while the user is working. However, if you’ve assigned them as _available_, then the user can simply launch the Company Portal and choose which apps to install.

* * *

The complete build
------------------

While not directly related, understanding how user assignments work alongside Group Tags are a key piece to a dynamic device build. It allows you to ensure your baseline components are installed up front while still allowing a unique experience for your end users, based on their job requirements.
