---
author: GuestUser
date: Thu, 16 Jul 2020 01:46:02 +0000
description: 'There are several settings within Azure and Intune/MEM that will dictate
  when users have administrative privileges. One of the primary options is to configure
  a setting within Autopilot – when we create an Autopilot profile, we assign it to
  devices that are registered in the tenant.'
slug: administrator-privileges-on-azure-ad-joined-machines-4lFva
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/administrator-privileges-on-azure-ad-joined-machines-4lFva_thumbnail.jpg
title: Administrator Privileges on Azure AD Joined Machines
---

There are several settings within Azure and Intune/MEM that will dictate when users have administrative privileges. One of the primary options is to configure a setting within Autopilot – when we create an Autopilot profile, we assign it to devices that are registered in the tenant. One of the options in the profile is the “User account type” setting, which we can set to Administrator or Standard:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1594862610809-1XWEN8RMN920AN4O8EBU/image-asset.png)

## The One Rule
---

A lot of organizations have a simple setup when it comes to Autopilot: All devices that are registered will aggregate into a dynamic group, and that dynamic group will be assigned the Autopilot profile pictured above. Here are the rules of the dynamic group which we’ll simply call “Autopilot Devices”:

```
(device.devicePhysicalIds -any _ -contains "[ZTDId]") 
```

This dynamic rule will gather ALL devices that are registered for Autopilot – this is because of the way the objects are stored in Azure, and all Autopilot devices contain the ‘ZTDId’ property within their multi-string array.

We can assign the “Z0t AutoPilot Profile” to the Autopilot Devices group, which we’ll call “MDM AutoPilot”, and we can even assign policies and applications to this group later on (there are, however, exceptions where certain policies and applications should be assigned to user groups instead).

Let’s look at another scenario – the majority of users at an organization will have Standard account types on their machine, but a specific group of users need to have Administrator accounts. Let’s call this special group the Developer team.

In our Autopilot device list, there is a property we can add to devices called the Group Tag. In the case of the Developer team, lets mark one of the machines with the group tag “dev”.

![2020-07-15 21_33_11-Administrator Privileges AADJ 1.pdf - Adobe Acrobat Reader DC.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1594863208320-FSM5AZ1OUV0U7F47SCEB/2020-07-15+21_33_11-Administrator+Privileges+AADJ+1.pdf+-+Adobe+Acrobat+Reader+DC.png)

Alongside our “Autopilot devices” dynamic group, we can create a second dynamic group that specifically gathers devices with the “dev” Group Tag. Here is the dynamic rule, which we’ll apply to a new group called Developer Team:

```
(device.devicePhysicalIds -any _ -contains "[OrderID]:dev") 
```

Now that we have our second device group, which we’ll call “MDM AutoPilot – Dev”, we can create a second Autopilot profile specifically for these tagged devices. This profile will instead have the “User account type” property set to Administrator. We will then assign this profile to “MDM AutoPilot – Dev”.

But there’s one more step – the original profile is assigned to ALL Autopilot devices. Therefore, we need to exclude our “MDM AutoPilot – Dev” device group from the original Autopilot profile. Here is what both profiles will look like in the end:

![First Autopilot profile – all devices included, but dev-tagged devices are excluded](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1594863320191-OPSX2YH6ZRXV60V787YZ/2020-07-15+21_35_03-Administrator+Privileges+AADJ+1.pdf+-+Adobe+Acrobat+Reader+DC.png)

First Autopilot profile – all devices included, but dev-tagged devices are excluded

![Second Autopilot profile – only dev-tagged devices are included](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1594863417683-VE31NFZ097DALRHQ8L31/2020-07-15+21_36_28-Administrator+Privileges+AADJ+1.pdf+-+Adobe+Acrobat+Reader+DC.png)

Second Autopilot profile – only dev-tagged devices are included

Additional Administrator Settings
---------------------------------

The above solution will work to separate Administrators and Standard users at the time of enrollment. If you need to specify any additional helpdesk/IT staff that should have administrative rights on devices, there is a global setting that can be configured in Azure.

Navigate to portal.azure.com -> Azure Active Directory -> Devices – Device Settings. Modify the “Additional local administrators on Azure AD joined devices” setting and add the appropriate users.

![2020-07-15 21_38_17-Administrator Privileges AADJ 1.pdf - Adobe Acrobat Reader DC.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1594863521138-QPFTNR37JABCHDUZY30Y/2020-07-15+21_38_17-Administrator+Privileges+AADJ+1.pdf+-+Adobe+Acrobat+Reader+DC.png)

This is a global setting that will add the specified users to the Administrators group on ALL Azure AD joined machines. These users will also be Administrators on their own machines even if the Autopilot profile is set to Standard user account type. One other exception to note – all Azure Global Admins will be administrators on their machines as well, despite the Autopilot profile settings.

An Alternative Custom Policy
----------------------------

Let’s consider one more scenario… let’s say you are unable to predict ahead of time which machines will land in the developers’ hands. If a machine is given to a developer last minute, you would need to quickly ensure the group tag is added to the device in the portal. This may not be viable if an Intune administrator is not immediately available to update the autopilot device.

Instead of relying on the group tags, we can create a custom policy that will ensure the administrative rights are always given to the developers - regardless of which machine they sign in to.

**NOTE:** If you create the policy and assign it to a user group that contains the developers, they will eventually have to reboot the machine to gain the administrative rights back. This is because the policy will hit the machines AFTER the Autopilot profile, and you will likely have your Autopilot profile set to Standard account type for all devices.

Let’s take a look at how to build the custom policy. Here is the OMA-URI and the string value, which is copied from an xml document, that will need to be configured:

![2020-07-15 21_40_57-Administrator Privileges AADJ 1.pdf - Adobe Acrobat Reader DC.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1594863685148-XDLKFI7FFIKK63TXSXK1/2020-07-15+21_40_57-Administrator+Privileges+AADJ+1.pdf+-+Adobe+Acrobat+Reader+DC.png)

The example above contains the full usernames – you can alternatively use the SIDs of the users within the quotation marks. While I did not include it this example, every Azure AD joined machine by default contains the Administrator account and two additional SIDs that represent the Azure Global Administrator and the Device Administrators (aka the users specified in the Azure device settings). You should include the Administrator and the two SIDs in this policy.

You can now assign the policy to the appropriate user group, and you can create additional versions that apply to different user groups.

**Additional NOTE:** The other interesting piece about this policy is unlike the Azure device settings, you can actually add the SIDs of the users. Since you can use a SID, that means you can use the SID of a GROUP instead of just an individual user. However, the ability to use group SIDs will **not work** unless you are on **Win10 build 2004 or higher**. If you wish to use a group SID, you can retrieve the SID from Active Directory or search the group’s object ID in the Microsoft Graph.
