---
author: steve@getrubix.com
date: Thu, 13 Jun 2024 15:53:23 +0000
description: '"Arguably the biggest change with Autopilot device preparation (APV2)
  is the technical flow and the emphasis on the user assignment. This is critical
  to understand as when we look at the trade-offs in the out-of-box experience (OOBE)
  compared to Autopilot V1 (APV1), they are not arbitrary; it all"'
slug: how-to-configure-autopilot-device-preparation-part-3-the-new-flow
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/how-to-configure-autopilot-device-preparation-part-3-the-new-flow_thumbnail.jpg
title: How to configure Autopilot Device Preparation Part 3 The New Flow
---

Arguably the biggest change with Autopilot device preparation (**APV2**) is the technical flow and the emphasis on the user assignment. This is critical to understand as when we look at the trade-offs in the out-of-box experience (OOBE) compared to Autopilot V1 (**APV1**), they are not arbitrary; it all has to do with the flow.

Quick recap
-----------

I won’t go too deep into the original flow, as I have done that here: [Autopilot Group Tags: Part 4 — Rubix (getrubix.com)](https://www.getrubix.com/blog/autopilot-group-tags-part-4?rq=group%20tags)

Basically, in APV1 you registered the PC to the Azure tenant prior to deployment, or even having the device in hand. This was done via collecting the device [hardware hash](https://learn.microsoft.com/en-us/autopilot/add-devices#collect-the-hardware-hash). Once the PC booted up and connected to a network, it would automatically reach out to check in with Microsoft to see if it was registered in Autopilot. If it was, the end user at that point would immediately see a custom OOBE, hiding most of the screens we looked at in my [previous post](https://www.getrubix.com/blog/how-to-configure-autopilot-device-preparation-part-2-user-experience).

New flow, who dis?
------------------

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/fbacea75-5f13-4097-ae94-029319f0e60b/APV2+Flow+2.png)

The flow for APV2 is based on user assignment, which means the device behavior during the first boot is left untouched.

Let’s breakdown this high-level overview.

### 1: Power on

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/02e0be31-7215-4005-94d5-ba0550b50be9/APV2+Flow.png)

In the first step, the end user powers on the device. There is nothing unique or different going on here than any other standard Windows Pro (or Enterprise) device. The user will have to complete the setup screens until they are asked to sign in.

> _To see all the screens and the steps the user must perform at each step, once again, check out the previous post_ [_here_](https://www.getrubix.com/blog/how-to-configure-autopilot-device-preparation-part-2-user-experience)_._

### 2: Sign in

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/a5d8df90-9aca-450d-baa3-882ffd39a243/APV2+Flow+%281%29.png)

When prompted, the user will sign in with their Microsoft 365 credentials (most likely their email address). Once they are authenticated, and assuming they are part of the APV2 assignment group, Intune will see that and begin to push out the Autopilot profile.

### 3: Join the device group

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/5756ace3-887e-4d27-8c9c-8296dda75a2c/APV2+Flow+%283%29.png)

One of the first things the Autopilot profile does is place the device in the designated Entra group. This should be the same group that configured apps, policies, and scripts are assigned to.

### 4: Provisioning time

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/bc8a3ffd-27c5-4b3b-b4ea-12b9c1baa448/APV2+Flow+%282%29.png)

Now that the device has been added to the Entra group, Autopilot will then proceed to have Intune push the required components to the device, which are monitored during the setup screen.

It’s all about the user
-----------------------

That’s the whole flow. As I said before, everything this time hinges on the end user. Nothing Autopilot or Intune related do not occur until after step 2 when the user signs in.

Hopefully this helps with understanding the primary differences between these two flavors of Autopilot. Remember; V1 isn’t going away anytime soon, especially while we wait for V2 to finish growing.
