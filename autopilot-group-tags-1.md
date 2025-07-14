---
author: steve@getrubix.com
date: Fri, 11 Mar 2022 19:04:13 +0000
description: 'Autopilot allows Windows devices to register to an Azure AD / Endpoint
  Manager environment prior to deployment. The PC is shipped directly to an end user,
  powered up, and when they sign-in, Windows is provisioned with applications and
  policies from Endpoint Manager.'
slug: autopilot-group-tags-1
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-group-tags-1_thumbnail.jpg
title: Autopilot Group Tags Part 1
---

Autopilot allows Windows devices to register to an Azure AD / Endpoint Manager environment prior to deployment. The PC is shipped directly to an end user, powered up, and when they sign-in, Windows is provisioned with applications and policies from Endpoint Manager. The device is production ready without IT having to manually configure the hardware first.

You might be thinking to yourself, “Steve- we already know this… have you been away from writing for so long that you’ve forgotten the basics"?” I appreciate the concern, but I’m getting to a point here. I would venture to say that many organizations group Autopilot devices with the Microsoft official “All Autopilot” dynamic membership rule:

```
(device.devicePhysicalIds -any _ -contains "[ZTDId]")
```

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/c6ba0471-8535-4a8f-9eed-d5b3711379f2/All+Autopilot.png)

So now if you use this group for assigning apps and profiles to your Autopilot fleet, it’s all or nothing. But how can we place the device in a specific group once it’s registered? I suppose you could designate a resource to monitor Autopilot devices all day, and when a new one comes in, they can place it in the right group. Or, we can leverage Autopilot Group Tags.

## What are Group Tags?
---

The Group Tag is an attribute that is added to the device during Autopilot registration. You can add it when running the PowerShell script to generate the hardware hash if uploading yourself like this:

``` pwsh
Get-WindowsAutopilotInfo -GroupTag <YOUR TAG NAME> -Online
```

Or you can simply provide your hardware vendor or partner with your tag names. The important part is that the tag is there during registration. The tag will show in the Endpoint Manager enrolled device list:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/d5057a33-b48a-409f-91b8-2218312965d1/Screenshot+2022-03-11+135800.png)

## Choosing a Tag
---

A Group Tag can be anything you’d like. It is a string value, so you can choose the words of your liking. However, I recommend we apply a structure to the tags. In my test lab and for most customers I work with, we use this schema:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/d2815be6-242e-43a9-b25c-390f04baf50e/schema.png)

Kind of looks like an eye exam…

Having tag structure helps when you consider the purpose is to have multiple tags for different kinds of device builds. You can even organize other device groups (non-Windows) with the same schema. Here are some examples we would use:

-   `WIN-AP-KIOSK` (Windows Autopilot Kiosk)
    
-   `IOS-BYOD-SALES` (iOS Bring-Your-Own Device Sales)
    
-   `AND-DEM-TRNG` (Android Device Enrollment Manager Training)
    
-   `WIN-CMG-CORP` (Windows Co-managed Corporate)
    

## How does the Group Tag help with assignment?
---

Azure AD groups that are used for Intune targeting can have dynamic membership rules to automatically populate the group with devices based on a variety of attributes. One of those attributes is the `devicePhysicalId`. What’s that? Well, that is the Group Tag; or at least what it’s actually called on the backend.

So now, we can create dynamic device groups that are looking for our Group Tag, or _tags_ so we can start creating some inheritance (more on that later).

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/77e97fac-ea3f-4fc6-aaf1-222eb319f0cb/WIN-AP-CORP.png)

So take a look at the device group named `WIN-AP-CORP`- the rule is set to look for tags that are EQUAL to `WIN-AP-CORP`. As soon as my device is registered with the tag, it will aggregate to that group. But why the `ZTD-AP` group up above? Glad you asked.

As I have multiple device builds, I still want a way to group everything that is registered and tagged in Autopilot, whether it be for reporting or just a _break-glass_ emergency assignment scenario. Here is another dynamic group:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/2dd8b77f-c1f6-43d3-a018-38278038b5c4/kiosk.png)

Whoa… inception groups?

In this example, the device is tagged `WIN-AP-KIOSK` and is obviously aggregated to the `WIN-AP-KIOSK` dynamic group. But now look at the overarching group, `ZTD-AP`. It is not looking for a tag _EQUAL_ to a value. It is looking for tags `CONTAINING_ “ZTD-AP”` . Because of this, it will collect anything `containing ZTD-AP`, such as `ZTD-AP-CORP` and `ZTD-AP-KIOSK`.

What does this mean for assignments? Well, have a look:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/31b24e31-8c86-4058-9380-ba6e45959123/final.png)

At the top level, I only assign what every device should receive in terms of applications and policy. Those will be inherited by the specific CORP and KIOSK groups. Now, you may notice redundancy. For example, take a look at the Microsoft Teams application assigned to both the CORP and KIOSK build. Why wouldn’t I just assign those at the top layer? Well, because perhaps I have an additional build and I don’t want Teams deployed, or maybe I’m testing Teams on the KIOSK build but the business hasn’t decided if we’re going to keep it yet.

The goal is to have the flexibility to deploy components as needed without blasting everything out to _All Devices_ and backing myself into an “exclude” corner. And with Group Tags, all of our builds will provision this way without me having to manually assign devices to groups when they’re registered.

But what about user assigned apps and policy, or what happens if I’m deploying multiple builds internationally? Stay tuned…
