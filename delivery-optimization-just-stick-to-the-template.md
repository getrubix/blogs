---
author: GuestUser
categories:
- intune
- security
- powershell
date: Fri, 02 Feb 2024 18:07:13 +0000
description: 'Here’s a short and sweet post on something that drove me crazy for over a few weeks. A little silly it took me this long, but the timing around some legitimate M365 Apps suite issues made it more confusing.'
slug: delivery-optimization-just-stick-to-the-template
tags:
- configuration profiles
- security
- script
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/delivery-optimization-just-stick-to-the-template-H4F8F_thumbnail.jpg
title: Delivery Optimization - Just Stick to the Template
---

Here’s a short and sweet post on something that drove me crazy for over a few weeks. A little silly it took me this long, but the timing around some legitimate M365 Apps suite issues made it more confusing.

## The Backstory
---

 Around the middle of November, I had several customers reach out to me about M365 Apps not deploying correctly during Autopilot. I saw some whispers online in MDT-related forums of a similar issue, possibly something due to a bad detection rule or an extra reboot flag, but nothing official from Microsoft. Some folks were saying updating to Win11 22H2 had fixed it, others could not resolve it.

At that point my suggestion was to either keep the suite out of ESP, or worst case try the win32 method with a setup.exe and xml. After a couple of weeks however, three out of my four customers who initially reached out were not having the issue anymore. The fourth customer, however, was still having intermittent issues.

## The Issue
---

I was building out a new Autopilot configuration with all new policies, scripts, and app packages; the policies were based on the Microsoft security baselines, though I never deploy the actual baseline profile itself. I prefer to create everything as configuration profiles (primarily settings catalog), which gives me more control over assignments/exclusions and helps avoid overall conflicts.

 I built the M365 App Suite, Autopilot Branding, and several other packages in the usual fashion I normally do for customers. When it came time to enroll, ESP would always time out with the Office CSP showing as ‘Not processed’.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/526e5cb7-a0b1-44e2-8eab-039134e7d0da/Picture1.png)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/d4e0eac4-11e8-41b6-954b-a48797ded651/Picture2.png)

Interestingly, if I hit try again, the ESP would actually complete about 10 seconds later and then restart the device. That’s good I guess, but definitely not the experience I want for end-users.

## The Cause
---

 I assumed it was lingering issues with M365 Apps on Microsoft’s end :). But - I ultimately started troubleshooting by excluding several policies at a time. In the end, it turns out it was the settings catalog profile for delivery optimization I had created.

##  WHAT?!
---

 I have always used the template version of DO without issues, and I presumed that doing the catalog format would behave exactly the same. However there is one specific policy that I noticed was different regarding the expected values:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/84753bcc-6056-48ee-88a3-e1782928c756/Picture3.png)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1ba9c619-4c75-4704-bede-499ba6153fdf/Picture4.png)

You would think that the settings catalog version of a policy should utilize the same CSP in the background as the corresponding template, so perhaps the settings catalog is incorrectly labelled, or perhaps it attempts its own conversion? The settings catalog would always show “success”, so the overall outcome was strange.

Regardless, I did not spend any more time testing this - because I know the template profile always works, I ultimately switched back to using that for the customer. Hopefully this helps you with any similar Microsoft deployment issues.
