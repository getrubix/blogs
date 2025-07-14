---
author: steve@getrubix.com
date: Tue, 29 Oct 2024 17:15:31 +0000
description: '"In most cases, the hardest part of doing anything is getting started,
  and that’s certainly true for implementing Intune. More often than not, I see folks
  thwarted by what I call the “fringe case.”Don’t forget that I’m doing an exclusive
  video series to follow along with this series"'
slug: getting-started-with-intune-part-2-dont-build-for-the-fringe
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/getting-started-with-intune-part-2-dont-build-for-the-fringe_thumbnail.jpg
title: Getting Started with Intune Part 2 Dont Build for the Fringe
---

In most cases, the hardest part of doing anything is getting started, and that’s certainly true for implementing Intune. More often than not, I see folks thwarted by what I call the “fringe case.”

Don’t forget that I’m doing an exclusive video series to follow along with this series in the [GetRubix YouTube members channel](https://www.youtube.com/playlist?list=UUMOF6q8UjlE5AFO52ht-G_L6A)!

**What is the Fringe?**
-----------------------

Let’s pretend your organization has 5,000 PCs. Here’s a typical breakdown of machine types:

-   **Corporate laptops:** 2,450
    
-   **New hires:** 350
    
-   **Training kiosks:** 900
    
-   **Call center desktops:** 1,250
    
-   **Factory floor machines:** 50
    

Each of these setups has its own challenges when moving to modern management, but let’s assume we’ve completed the discovery steps outlined in Part 1 and have the right information. Here’s what we know:

-   **Corporate laptops:** Standard end-user devices, with most running the latest build of Windows 11, and the remainder on Windows 10 22H2. All were deployed during the last 3-year refresh cycle. Users are split between in-office and remote work.
    
-   **New hires:** Corporate expects to onboard 350 new remote employees over the next four months.
    
-   **Training kiosks:** Touchscreen all-in-ones running Windows 11 in single-app kiosk mode, solely for corporate training software. Users can’t access email, Teams, or other Microsoft 365 services on these.
    
-   **Call center desktops:** Mini-PC workstations functioning as thin clients that only run the corporate VDI solution. Domain-joined but with minimal group policy. Regular patching is the primary concern here.
    
-   **Factory floor machines:** Critical desktop workstations used for manufacturing, with no internet access.
    

Of these device types, which do you think will be the most challenging to move to Intune management?

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/b53c4c96-6396-4f34-86d8-e9467b9628c8/SCR-20241029-ltco.png)

Do you know where your devices fall on this matrix?

**High-Impact Quick Wins**
--------------------------

If the fringe is the most challenging, your easiest target should have low friction and high impact.

For example, if issuing laptops to new remote hires is currently difficult, and there’s pressure to improve this process, that’s a great place to start. Imagine someone gets hired, and instead of procuring, imaging, domain-joining, and configuring a device, you simply drop-ship it to their location. That’s a win.

These areas are often “low effort” because remote workers usually rely on cloud-based applications and solutions, making Intune setup quicker.

**Starting with the Right Target Matters**
------------------------------------------

This isn’t just about easy vs. hard; it’s about what Intune implementation looks like to your organization. Starting with the fringe case can make Intune implementation appear complex and challenging, giving the impression that Intune isn’t ready for the organization, potentially stalling adoption. And trust me, no one is going to say, “Well, that was tough, but let’s try it with a different business unit.”

On the other hand, if we start with a low-effort, high-impact area, here’s what happens:

-   IT will have an easier time supporting this use case.
    
-   The business will see a quick win with Intune and Autopilot.
    
-   Other units will be more likely to adopt it.
    
-   Intune will be viewed as a success.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/feeb332e-6d19-44bd-a4d2-db8d78583c90/SCR-20241029-ltes.png)

Plot out your use cases to determine their level of effort vs the positive impact on the business.

**No PC Left Behind**
---------------------

Choosing not to start with the fringe case doesn’t mean ignoring it. In my experience, obstacles tying these machines to legacy management gradually change. Specialty programs that once required on-premises hosting move to the cloud, and as resources transition to Azure, domain dependency decreases.

By the time the rest of the business is onboarded to Intune, tackling the fringe case becomes less daunting. Does this mean we’ll always be able to transition every device to modern management? No. Sometimes, the fringe case remains managed on-premises or with another tool like SCCM.

But that doesn’t mean we don’t try, and it certainly doesn’t mean we hold back the rest of the fleet from moving to Intune. Starting with the “quick win” allows you to pave a path forward for the entire environment.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/59770f6c-7fc0-4232-8371-7e9b6b2aa014/SCR-20241029-ltfz.png)

Regardless of where your devices fall on the grid, you can make a plan that includes the whole fleet.
