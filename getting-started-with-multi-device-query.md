---
author: steve@getrubix.com
date: Sat, 22 Feb 2025 22:40:11 +0000
description: '"Intune device query was a big step forward when it came out last year.
  The ability to use KQL to grab real-time information from a PC was a much welcome
  feature for the platform, even if it was only available in the Advanced Analytics
  add-on.But one of"'
slug: getting-started-with-multi-device-query
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/getting-started-with-multi-device-query_thumbnail.jpg
title: Getting Started with Multi-Device Query
---

Intune device query was a big step forward when it came out last year. The ability to use KQL to grab real-time information from a PC was a much welcome feature for the platform, even if it was only available in the [Advanced Analytics](https://learn.microsoft.com/en-us/mem/analytics/advanced-endpoint-analytics) add-on.

But one of its main criticisms was the fact that you could only query information from a single device at a time. And to be fair, that was problematic. Think about a production environment— if you’re looking for a malicious file or TPM status, chances are you want to get that information for your whole fleet.

Well now you can. The new Device Query is available and you have the ability to run queries against all of your enrolled Windows devices.

_If you’re a member of our YouTube channel, you can watch my video on the subject here:_ [_https://youtu.be/iHKVakzzGbQ_](https://youtu.be/iHKVakzzGbQ)

The boring stuff
----------------

To use Device Query in your tenant, make sure you have the following covered:

## Pre-requisites
---

- ### Licensing

  - You need a license that includes Advanced Analytics, so that means one of the following:
    
  -   `Intune Suite`
        
  -   `Advanced Analytics add-on`
        

- ### Device

  -   `The device must be corporate owned and Intune managed (duh)`
    
  -   `Device must be enrolled in Endpoint Analytics` (read [here](https://learn.microsoft.com/en-us/mem/analytics/enroll-intune) if you’re not sure how to go about that)
    

I think that covers it. Let’s go!

## Where is Device Query?
---

The single device query is still located within the managed device menu for each PC. To access the tenant-wide query, log in to [https://endpoint.microsoft.com](https://endpoint.microsoft.com) and navigate to **Devices > Device query**:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/f01b3962-b4c0-4091-87d1-b5c91b9b3411/Pasted+Graphic.png)

That was easy

> You’ll notice the banner on the top trying to tell me my Advanced Analytics trial has ended. I’m not too sure what this is about, since I have purchased the Intune Suite licenses and applied them to my users. Hoping this is just a hiccup, but figured I’d mention it in case you see the same. Everything has been working fine.  

## Device Query Layout
---

Don’t worry if you’re not super familiar with KQL—it’s a pretty straightforward ‘language’ to pick up, and there are a ton of amazing resources online. One of them is [https://www.kqlsearch.com/devicequery](https://www.kqlsearch.com/devicequery) by the incredible Ugur Koc.

But before that, let’s go over the lay of the land, so to speak. Here is a break down of the device query page:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7f04e113-fc65-426a-aa9f-5d6987d5ed30/SCR-20250222-nwpx.png)

-   A: These are the categories of attributes you can use in your device queries
    
-   B: Specific category selection
    
-   C: List of attributes within a category that can be queried
    
-   D: Query builder; this is where you actually type out the KQL
    
-   E: Results from the query are shown here
    

## KQ-what?
---

In the above image you can see the KQL query we’re using is:

```
Device 
| where EnrollmentProfileName contains "Default M365"
| project DeviceName, EntraDeviceId, EnrollmentProfileName, LastSeenDateTime
```

In human speak this means we are looking for any device with an enrollment profile containing the term “Default M365”. When we locate them, we want to return their name, Entra ID, enrollment profile name, and the last date and time they were seen online.

If we break this down, we can see that we start with **Device**, which is the category of attributes we’re looking for. Throughout the query you’ll the attributes from **Device** spread throughout including **EnrollmentProfileName, DeviceName,** and **EntraDeviceId**.

To make the query work, you need two elements: the `where` and `project` clauses.

-   **where Clause:** Acts as a filter to narrow down the dataset based on conditions. It works like SQL’s WHERE clause, allowing you to specify conditions such as string matches, numeric comparisons, or DateTime filters.
    
-   **project Clause:** Selects specific columns to display, reducing unnecessary data in results. It’s similar to SELECT in SQL.
    

So we want to look at the `Device` category for objects `where` `EnrollmentProfileName` contains the phrase “Default M365” (which, in my case, is “Default M365 Corp Profile”) and `project` the columns `DeviceName`, `EntraDeviceId`, `EnrollmentProfileName`, and `LastSeenDateTime`.

## You’ll get used to it
---

There’s more we’ll get into, but for now, if this is your first time diving into KQL, take what we’ve done here and try to build your own queries. Start simple and watch as your queries return the attributes you want. I promise you’ll get comfortable with it.

Next time, we’ll take a look at more complex queries that contain attributes from multiple categories.
