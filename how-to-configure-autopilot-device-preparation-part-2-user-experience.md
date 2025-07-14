---
author: steve@getrubix.com
date: Tue, 11 Jun 2024 18:50:40 +0000
description: '"In part 1 we looked at the step-by-step process for configuring the
  Autopilot device preparation policy. Today, we’ll look at the end user experience
  and the ways this experience differs from Autopilot V1. I’ll also show you how critical
  Windows Pro VS Enterprise is for this version of"'
slug: how-to-configure-autopilot-device-preparation-part-2-user-experience
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/how-to-configure-autopilot-device-preparation-part-2-user-experience_thumbnail.jpg
title: How to configure Autopilot Device Preparation Part 2 User Experience
---

In [part 1](https://www.getrubix.com/blog/autopilot-device-preparation-part1) we looked at the step-by-step process for configuring the Autopilot device preparation policy. Today, we’ll look at the end user experience and the ways this experience differs from Autopilot V1. I’ll also show you how critical Windows Pro VS Enterprise is for this version of provisioning.

> _You can follow along with this guide using either a physical PC or virtual machine. For info on setting up Hyper-V machines to use for testing, check my write-up on it here:_ [_Hyped-up Hyper-V — Rubix (getrubix.com)_](https://www.getrubix.com/blog/hyped-up-hyper-v?rq=hyper-v)

Boot up and get started
-----------------------

Start up your physical PC or virtual machine and wait until you see the _“Choose your country or region”_ selection screen. To verify what version of Windows you’re using, type **fn+Shift+F10** to open the cmd prompt and type `winver` and hit return.

![1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718130499008-850Z61VLUYJQ1YO9LXB3/1.png)

![1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718130499008-850Z61VLUYJQ1YO9LXB3/1.png)

![2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718130498969-DDNPGPRP0SSQB37T0W7R/2.png)

![2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718130498969-DDNPGPRP0SSQB37T0W7R/2.png)

#block-yui\_3\_17\_2\_1\_1718130455724\_13392 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -20px; } #block-yui\_3\_17\_2\_1\_1718130455724\_13392 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 20px; margin-bottom: 20px; }

This will show you two important things:

-   First, if you’re using Windows 11 Pro or Enterprise. This will affect the screens you will see during the out-of-box experience (OOBE)
    
-   Second, you will see the Windows build number. The minimum requirement for Autopilot device preparation is 22631.3374 (Windows 23H2) or 22621.3374 (Windows 22H2). I have personally only tested 23H2 and the 24H2 insider build.
    

Choose your region and click **yes**.

On the next screens, choose your keyboard type and then decide whether to add a second keyboard or not (I have chosen to hit **Skip**).

![3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718130548025-P30ISP4QJYVQNK52FQ3T/3.png)

![3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718130548025-P30ISP4QJYVQNK52FQ3T/3.png)

![4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718130548007-EP4MICI24VZ6WZDZM0Y8/4.png)

![4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718130548007-EP4MICI24VZ6WZDZM0Y8/4.png)

#block-yui\_3\_17\_2\_1\_1718130455724\_22442 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -20px; } #block-yui\_3\_17\_2\_1\_1718130455724\_22442 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 20px; margin-bottom: 20px; }

No longer hidden
----------------

Now we come to three screens that were hidden in Autopilot V1. This is due to the flow of V2 and the fact that the hardware is no longer pre-registered to the tenant. In fact, the Autopilot profile in this version won’t come down to the device until after the user signs in, so we’re working with a pure, OOBE.

> _Keep in mind, two of these screens can be skipped by using Windows 11 Enterprise._

### End user license agreement (EULA)

This page will be shown for both Windows 11 Pro and Enterprise versions.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/da36bc2a-5d92-4b29-8843-4992f0cfe8ad/5.png)

-   Click **Accept** on the License Agreement page.
    

### Device name

This will only be shown in Windows 11 Pro, however it is the most problematic given that in most situations, an organization will NOT want an end user to name their own PC.

If need be, the computer name can be changed again via a PowerShell script deployed through Intune after enrollment.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/bfe5ac73-710d-486b-8b28-ef1fc50ad97d/6.png)

-   We will click on **Skip for now**.
    

### Personal or work/school

Probably one of the most critical OOBE screens, this will also not be shown in Windows 11 Enterprise. Here, the end user can choose _Set up for personal use_ or _Set up for work or school_.

In order for Autopilot V2 to work, you need to select the work/school option. Otherwise, the PC will be expecting a consumer Microsoft account and not an Entra ID.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/a36518a9-c3c7-425b-8f7c-cd160018b7b0/7.png)

-   To continue with Autopilot enrollment, choose **Set up for work or school**
    
-   Click **Next**
    

On the sign-in screen, the PC is expecting an enterprise or education account.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ac71a2d5-f416-4a49-b945-4e77dee5167b/8.png)

-   Enter your M365 credentials and click **Next**
    

Once you enter your email, you should see the corporate logo on the next screen.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/95402276-2065-4965-b036-845f3f0446c9/9.png)

-   Enter your password and click **Sign in**
    

We are now in Autopilot
-----------------------

Assuming the user you signed in with is part of the group that is assigned the device preparation profile, you will start seeing the setup screen and the new status page.

### Start the setup

![10.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718131386265-8T070QEOXGON0DPW642W/10.png)

![10.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718131386265-8T070QEOXGON0DPW642W/10.png)

![11.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718131386265-RNACALKKAFGB6H6KVBIP/11.png)

![11.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718131386265-RNACALKKAFGB6H6KVBIP/11.png)

#block-yui\_3\_17\_2\_1\_1718130455724\_39695 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -20px; } #block-yui\_3\_17\_2\_1\_1718130455724\_39695 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 20px; margin-bottom: 20px; }

The new status page is quite different, only showing the user the overall setup progress of the PC.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/07bf861d-1042-463f-8ad1-4643e8c395ad/12.png)

So far from my testing, it seems like the numbers correlate to the following:

-   1% = Intune management extension has started installing
    
-   3% = Apps and scripts start installing
    
-   3-100% = profit?  
    It looks like it just jumps to 100% over the next few minutes depending on what you’re installing, stopping at random percentages along the way. I’m not really sure if there’s any deeper tracking happening here until it is complete.  
    \[IMAGE 13\]
    

Once the required setup is complete, click **Next**

### This is private

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/a1130bdf-aa60-4e4f-9750-7d591c67d516/14.png)

The last screen that is no longer skipped with Autopilot is the device privacy settings. These can also be configured through Intune and applied to the device after enrollment.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/05cec7c2-94ab-4ea1-b997-c72cbbac571c/15.png)

For now, just scroll to the bottom of the settings and click **Accept.**

Looks like we made it
---------------------

After the privacy settings, the device should start preparing the user profile. Once completed, you will now be on the fully enrolled and managed desktop.

![16.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718131574450-NBWZJF8P9CCXY6FU46QX/16.png)

![16.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718131574450-NBWZJF8P9CCXY6FU46QX/16.png)

![17.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718131574508-C9EIDHCP39XTOZNJKCH0/17.png)

![17.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1718131574508-C9EIDHCP39XTOZNJKCH0/17.png)

#block-yui\_3\_17\_2\_1\_1718131490199\_24338 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -10px; } #block-yui\_3\_17\_2\_1\_1718131490199\_24338 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 10px; margin-bottom: 10px; }

That’s it for today. There are definitely more screens visible in the OOBE compared to original Autopilot, but that seems to be the trade-off for not requiring hardware registration.

In the next part, we’ll zoom out and look at the actual flow of device preparation and what’s happening behind the scenes.
