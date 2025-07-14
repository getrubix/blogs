---
author: steve@getrubix.com
date: Tue, 28 Jul 2020 13:23:14 +0000
description: '"I''ve written a lot about the Enrollment Status Page (ESP) in the Microsoft
  Endpoint Manager (here is the most recent), mainly because it can be a tricky thing
  to nail down.&nbsp; My latest issue with it has been a drastic increase in the time
  it takes for devices"'
slug: multiple-enrollment-status-pages-a-psa
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/multiple-enrollment-status-pages-a-psa_thumbnail.jpg
title: Multiple Enrollment Status Pages A PSA
---

I've written a lot about the Enrollment Status Page (ESP) in the Microsoft Endpoint Manager (here is the [most recent](https://www.getrubix.com/blog/please-wait)), mainly because it can be a tricky thing to nail down.  My latest issue with it has been a drastic increase in the time it takes for devices to get through the "Preparing your device for mobile management" step:

![2020-07-27 14_51_30-ZTD-AP-EUR-1909 - VMware Workstation.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595941042596-FVH7MG1PLSRM6SQKNKN6/2020-07-27+14_51_30-ZTD-AP-EUR-1909+-+VMware+Workstation.png)

At first, my team chalked this up to general slow down on the Azure side, but then soon noticed the behavior wasn't consistent across other tenants we work in.  Then we remembered the update in June with the 2006 release we implemented- targeting ESPs to device groups.  Here is the official release note:

![2020-07-28 08_59_04-What's new in Microsoft Intune - Azure _ Microsoft Docs and 21 more pages - Pers.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595941176064-LVEHDAR8JOK50E11VW7M/2020-07-28+08_59_04-What%27s+new+in+Microsoft+Intune+-+Azure+_+Microsoft+Docs+and+21+more+pages+-+Pers.png)

Now this is a very welcome update as it allows us to set different criteria during device provisioning based on the device group, which ultimately is based on a dynamic group that implies a build flavor (engineering, accounting, HR, etc.)  We've passed this feedback onto Microsoft for over a year now and it is always great to see things that customers need implemented in production.  Naturally, we were eager to test.

Configuring what the ESP will wait for during provisioning is setup in [**https://endpoint.microsoft.com**](https://endpoint.microsoft.com) **-> Devices -> Enroll Devices -> Windows enrollment -> Enrollment Status Page:**

![2020-07-27 15_10_25-All users and all devices _ Properties - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595941242181-XJJCWSXIC0EDPU44G22P/2020-07-27+15_10_25-All+users+and+all+devices+_+Properties+-+Microsoft+Endpoint+Manager+admin+center.png)

In my lab tenant contains three distinct builds that that require different applications targeted at each one.  We created three ESPs with each one locking the [Autopilot Branding](https://github.com/mtniehaus/AutopilotBranding) package that sets many default preferences to the machine has to run prior to a user object being created.

![2020-07-27 15_16_16-ZTD-AP-EUR-1909 - VMware Workstation.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595941288638-25X1G9YX8D8BRN4PQ8PD/2020-07-27+15_16_16-ZTD-AP-EUR-1909+-+VMware+Workstation.png)

![2020-07-27 15_16_16-ZTD-AP-EUR-1909 - VMware Workstation.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595941288638-25X1G9YX8D8BRN4PQ8PD/2020-07-27+15_16_16-ZTD-AP-EUR-1909+-+VMware+Workstation.png)

![2020-07-27 15_30_21-ZTD-AP-APAC-1909 - VMware Workstation.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595941296403-WPFOBNQWK04168V4XTB5/2020-07-27+15_30_21-ZTD-AP-APAC-1909+-+VMware+Workstation.png)

![2020-07-27 15_30_21-ZTD-AP-APAC-1909 - VMware Workstation.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595941296403-WPFOBNQWK04168V4XTB5/2020-07-27+15_30_21-ZTD-AP-APAC-1909+-+VMware+Workstation.png)

![Untitled picture.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595941296443-YVCHVSB3B6ZLFXH48QOU/Untitled+picture.png)

![Untitled picture.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595941296443-YVCHVSB3B6ZLFXH48QOU/Untitled+picture.png)

#block-yui\_3\_17\_2\_1\_1595940882928\_32993 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -9px; } #block-yui\_3\_17\_2\_1\_1595940882928\_32993 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 9px; margin-bottom: 9px; }

The logic worked as expected- each type of device received an ESP that only waited for the applications it was supposed to.  So what was the issue?  Time- what was usually a less-than 5 minutes process was taking closer to 20-25 minutes.  This was raising our overall time-to-desktop significantly, which is never a good thing.  But why was this happening?

You see, nearing the end of the **"Device Preparation…"** phase, the Intune Management Extension is installed along with the manifest telling the ESP which applications and policies to wait for.  With a default ESP, the wait is usually not a problem, but now that we have three configured, my guess is that there is some logic that has to be evaluated so the device knows which ESP it is supposed to follow.  This additional piece must be what was adding the time.

Testing was simple.  I went ahead and removed the additional ESPs that were created and re-enrolled my devices.  Just like that, things returned to normal.  **"Device Preparation…"** was back to under 5 minutes (about 3 minutes and 28 seconds on average, to be precise).

What’s the point?
-----------------

Well, I'm not really sure.  Try to take this as more of an FYI as opposed to a "don't do this"!  After all, the logic with multiple ESPs does end up working as it should, it was just the time that was unacceptable.  For now, we went back to an older method- configuring the default ESP to wait for all Autopilot Preference packages and assuming it would only force the device to wait for the apps targeted to it.  And this still works.

![esp.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595941411284-1FSULB9GHY56Y1YMREYA/esp.png)

Let me know your experiences and if you've seen any adverse results with the new ESP options.
