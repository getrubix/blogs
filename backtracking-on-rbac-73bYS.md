---
author: GuestUser
date: Fri, 06 Sep 2024 19:29:25 +0000
description: 'Don’t worry, this isn’t another discovered issue or service advisory.
  I wanted to put together some general info and recommendations around Intune’s role-based
  access controls (RBAC). I often get asked about the capabilities and how to support
  different scenarios, so I figured it would be good to list'
slug: backtracking-on-rbac-73bYS
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/backtracking-on-rbac-73bYS_thumbnail.jpg
title: Backtracking on RBAC
---

Don’t worry, this isn’t another discovered issue or service advisory. I wanted to put together some general info and recommendations around Intune’s role-based access controls (RBAC). I often get asked about the capabilities and how to support different scenarios, so I figured it would be good to list some critical points here.

## The Roles
---

Out of the box, Intune has some great built-in roles that you can directly assign to users. However, I always advise duplicating one of the examples and customizing them based on each desired role. There’s lots of great articles out there on how to go through the individual settings with screenshots from the console… I’m going to assume you already know where to find these settings.

I won’t get too specific on each individual permission that you can customize; at a high-level, I typically introduce customers to the process with three basic roles:

### Read-only role

-   I replicate this one from the built-in “Read Only Operator” role. There are still some categories where the **Read** and **View Reports** are not on by default, such as ASR/WDAC settings or granular service connectors. Whether you want to include these in the mix for a true read-all is up to you.
    

### Helpdesk role

-   I replicate this one from the built-in “Help Desk Operators” role. Aside from filling in any missing Read/View Reports rights, this role allows a number of remote actions on **Intune-enrolled** and **Cloud attached devices**. There are also some default Remote Help permissions and Managed/Mobile apps assignment capabilities enabled, as well as some enrollment program/sync settings disabled (which I think should be enabled – should always be able to sync lists and maybe even register devices for say Autopilot).
    
-   The key here is to decide how many Helpdesk tiers you need and what rights are allowed for each tier. Some organizations design their Tier 1 role with less remote actions and more of a read-only approach (albeit with some actions like device sync, restart, etc). Maybe Tier 2 can have most of the other remote actions like wiping or deleting device records – or maybe that’s for Tier 3. You may also want to separate the Remote Help elevation settings between different helpdesk tiers as well.
    

### ADMIN ROLE

-   For this one, I typically make a custom role where the majority of the rights are enabled. However, when it comes to global settings like connectors and enrollment program tokens, I set those to read-only - this way these limited “admins” shouldn’t be able to break something that affects everyone in the tenant.
    
-   Since this role will allow policy and app changes, we also want to limit any impacts by incorporating the **scope tags** into the assignments. I also recommend using the scope tags where applicable for the Helpdesk Role assignments, but some gotchas are detailed further down
    

Now, here’s some gotchas with the roles themselves:

1.  If you intend to have multiple granular roles, and you have a user that falls into more than one assignment, the rights will be **merged** for that user.
    
2.  A lot of people miss the fact that custom roles require the assigned admins to have an **Intune license**. This is different than a full Intune Administrator, as Entra ID technically lets you do unlicensed admin roles.
    
3.  If you make changes to the permissions, be prepared to wait a few hours for the changes to take effect on current users.
    

## The Scope Tags
---

With the high-level roles called out, we now need to limit where scoped admins can make changes in the tenant. This is why nearly all configurations in Intune can have a **Scope Tag** applied – when you create an assignment for a role, you can specify which devices and configurations are visible to the admin based on the selected scope tag (you can do more than one).

Let’s say I want to assign our buddy Dustin to the helpdesk role – however I only want him to be able to see devices and configurations relevant to his location in Texas (and therefore he can only perform remote actions on devices in Texas). I would do the following:

1.  I create a scope tag named “Texas” - I assign the dynamic device group(s) that would contain all devices in Texas.
    
2.  For any enrollment settings, policies, scripts, or applications that he needs to view for his location, I would add the Texas scope tag in the properties of each configuration.
    
3.  When I create an assignment on the custom role, I’ll prep a security group containing Dustin and proceed with the assignment and scope tag.
    
4.  When Dustin logs into the tenant, he’ll be able to see only the devices and configurations that have the matching Texas scope tag.
    

Now, here’s some gotchas with Scope Tags:

-   You can not have a scoped admin with one set of rights on one object, and then a different set of rights on another. This is because multiple role assignments ultimately have the rights merged for that user – even if you are using different scope tags for the separate role assignments, the rights and the scope tags end up being merged for that user.
    

-   If you have devices that receive shared configuration profiles, and those profiles do not contain the user’s matching scope tag, you will **not see t**he settings summarized in the device’s configuration profile blade. I think this is a major flaw with the console; just because a policy is not scoped for someone, doesn’t mean they shouldn’t be aware that it’s being applied to their devices.
    

-   Because of the point above, and because of other administrative roles where you have to separate things by scope tag, you will need to carefully think about how you will handle shared configurations. Because of the reporting piece, I personally would recommend applying all applicable scope tags to the shared configurations – any administrators that have the rights to modify these profiles will need to be **educated** on the potential impact they may cause. Scope Groups can potentially help limit this impact, but there’s some gotchas there as well.
    

## The Scope Groups
---

When assigning a group to a custom role… before you select the scope tag, you have the option to customize the **Scope Groups**. For any roles that allow configuration assignments and remote actions, you can limit these tasks to users or devices that are part of the selected scope groups.

Sounds like it should limit assignment capabilities, but let’s jump straight into the gotchas:

-   The big problem with this setting is more so for admins who need to assign configurations over time. Let’s say I gave Dustin an admin role (instead of helpdesk, congrats Dustin) – he now has the capability to create and assign policies where he needs. However, I only want him to be able to assign things to users/devices in his location. I could specify a security group with all of the Texas users, as well as whatever security groups with the Texas devices. This should solve the problem, right?
    

-   It does for now… but what if Dustin now needs to assign a policy to only a subset of the Texas devices – can he make a smaller security group and only target those? If that new group is not added into the Scope Groups section of the role assignment, then the answer is **no**. The same thing goes for any smaller subset of users - an Intune Administrator (full or RBAC-based) will need to edit the role assignment for Dustin and add the new security group(s) into the Scope Groups section. Depending on how frequently Dustin needs to do this, you can see how tedious this can become for the higher-up Intune administrators.
    

-   Because of the scenario above, you may decide to choose the **all users** and **all devices** options for Scope Groups. If you do this, you must once again educate your admins to make sure they do not negatively impact any users or devices with configuration changes. If I am giving Dustin the “all users” and “all devices” option for Scope Groups, he can literally assign things to ALL users and ALL devices tenant-wide - **this means that the Scope Tag will NOT limit the impact of the “all” assignments.**
    

## Entra ID Rights
---

A lot of folks that test Intune RBAC for the first time are users who previously had full Intune Administrator rights. One of the first things they realize is they can no longer manage Entra ID device records – they also can’t retrieve the BitLocker recovery keys or LAPS passwords from the Intune console, since those are technically coming from Entra ID.

With this in mind, you could potentially use one of the built-in roles in Entra ID and assign it to the same group that is being used for the custom Intune role. Cloud Device Administrator would give you back all rights you previously had as an Intune Administrator – having said that, you may want to limit some of these rights with a custom Entra ID role.

And of course, make sure you remove the Intune Administrator role prior to testing the custom RBAC role – otherwise this will override any custom role configurations. Hopefully this gives you a good starting point If you’re just getting into RBAC for Intune.
