---
author: steve@getrubix.com
categories:
- intune
- azure
date: Fri, 26 Jul 2019 23:12:00 +0000
description: "When I have the standard “you don’t need domain join and SCCM if you use Intune” conversation, the first push-back from the uninitiated is that Intune does not have full parity with on-premise GPO (Group Policy Objects). This hasn’t been true for sometime, but now with the official release of Administrative Templates, it is even more ludicrous."
slug: yes-there-are-gpos-in-intune
tags:
- intune
- configuration profiles
- azure
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/yes-there-are-gpos-in-intune_thumbnail.jpg
title: Yes there are GPOs in Intune
---

When I have the standard “you don’t need domain join and SCCM if you use Intune” conversation, the first push-back from the uninitiated is that Intune does not have full parity with on-premise GPO (Group Policy Objects). This hasn’t been true for sometime, but now with the official release of Administrative Templates, it is even more ludicrous.

Types of Intune policy
----------------------

Policy from Intune can be divided into three categories: Configuration profiles, custom CSP, and Administrative Templates.

### Configuration profiles

These are the easy ones; profiles that are nothing more than GUI toggles in the console just like any standard MDM (Mobile Device Management) solution. Want to disable Cortana? Just hit the ‘block’ button in the console. Require CTRL + ALT + DEL to unlock a PC? Click away, my friend.

![2019-07-26-08_52_30-interactive-logon-microsoft-azure.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581033581292-R1I0LLHLBIC8CQAVUTEB/2019-07-26-08_52_30-interactive-logon-microsoft-azure.png)

### Custom CSP

So what happens when you want to set a policy that’s nowhere to be found in the Intune console? That’s when you head over to the holy grail of modern management, the Policy CSP page. Here you’ll find every policy that can be configured to Windows 10 through Intune. The one catch is you have to put them together yourself.

Let’s say for example you want to disable the consumer features of Windows that business love so much (that was sarcasm). First step is to find that policy on the [Microsoft CSP page](https://docs.microsoft.com/en-us/windows/client-management/mdm/policy-configuration-service-provider). As you can see below, the **AreaName** is “Experience” and the **Policy** is “AllowWindowsConsumerFeatures”. Our options are represented by the values 0 (Not Allowed) and 1 (Allowed).

![2019-07-26-08_56_00-policy-csp-experience-_-microsoft-docs-and-11-more-pages-microsoft-edge.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581033680907-H7400YVE2LOKJWB760RB/2019-07-26-08_56_00-policy-csp-experience-_-microsoft-docs-and-11-more-pages-microsoft-edge.png)

The basic construction of the policy is usually **./<Scope>/Vendor/MSFT/Policy/Config/<AreaName>/<Policy>**.

Scope can also be found on the page with the policy documentation. So in this case, our policy will read as **./Device/Vendor/MSFT/Policy/Config/Experience/AllowWindowsConsumerFeatures**. Intune has an option when creating configuration profiles to choose “custom”. Simply apply the values we aggregated from the CSP page to set and assign:

![Custom configuration profile values](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581033821022-4PICDANOTG77HV7BP6FN/2019-07-26-09_01_00-add-row-microsoft-azure.png)

Custom configuration profile values

### Administrative templates

Ah yes- this is the long awaited feature that was keeping so many of us nerds in suspense. Actual, GPOs in the Intune console. Simply add a configuration profile with the type “Administrative Templates”. From there, you will find an embarrassingly disorganized list of every ADMX policy available.

![All available GPOs- without any organization](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581034178585-OV9G9OHFC8GYZC2X3S71/2019-07-26-09_03_04-z0t-onedrive-policy-settings-microsoft-azure.png)

All available GPOs- without any organization

Let’s go ahead and set a configuration for OneDrive policy. After creating the Administrative Templates profile, open the settings and search for “OneDrive”. Everything available in that node should populate. Select the settings you want to configure, and just like traditional GPO, you’ll be presented with the options. That’s it. You can make as many of these profiles as you’d like as they pertain to different areas of Windows.

![Just like AD group policy](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581034258556-NVOXBLGHZ3CWG9KEPCY3/2019-07-26-09_04_14-silently-sign-in-users-to-the-onedrive-sync-client-with-their-windows-credential.png)

Just like AD group policy

There we have it; three different ways to add policy through Intune. Again, let me emphasize that this provides us complete control over a non-domain joined PC. And the advantage of these policies through Intune is unlike the local domain, these can be configured and enforced anywhere, regardless of the machines location.

Coming up next, I will tell you why you don’t need SCCM either.
