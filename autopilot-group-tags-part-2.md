---
author: steve@getrubix.com
categories:
- security
date: Sat, 12 Mar 2022 22:00:47 +0000
description: "In Part 1, we went over the basics of Autopilot Group Tags and how we can use them to target devices for application and policy provisioning upon registration. However, the focus was on the device build. Today we’re going to take that a step farther."
slug: autopilot-group-tags-part-2
tags:
- security
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-group-tags-part-2_thumbnail.jpg
title: Autopilot Group Tags Part 2
---

In [Part 1](https://www.getrubix.com/blog/autopilot-group-tags-1), we went over the basics of Autopilot Group Tags and how we can use them to target devices for application and policy provisioning upon registration. However, the focus was on the _device build_**_._** Today we’re going to take that a step farther. We are going to go through how to use Group Tags to provision devices based on build and _location_.

I’m going to break this up into two scenarios: one business that is grouping devices by a site location (Magic Coffee Co) and another that is grouping based on regions (Global Operations Inc). Of course, these are fabricated names, unless they exist, in which case feel free to use my Group Tag strategies.

Setting sights on sites
-----------------------

I have twin daughters turning 7 this month so I’m allowed to make terrible dad puns.

Magic Coffee Co has three corporate locations in the USA; New York, Chicago, and Los Angeles, each with about 300 users. All locations use at least two types of PC builds. The first is an end-user corporate laptop, with standard knowledge worker apps. The second is a shared desktop PC that is stationary and used in their call center.

The locations have their own set of policies, mapped network drives, shared printers, and in some cases, unique internal applications. Sites are managed by their own IT departments, so often are not on the same timeline in regards to Windows and Office updates.

Finally, the New York location, has one additional build, which is a kiosk used for new hire training. Magic Coffee is considering adding new hire training programs at the additional locations, but that decision will not be made right now.

Let’s recap what we have:

-   3 corporate locations
    
-   Chicago and Los Angeles use 2 PC builds
    
-   NY uses the same 2 builds, but with an additional 3rd build
    
-   All locations require access to unique resources
    
-   Locations are not on same software update cycle.
    

So what does this mean for our Group Tag strategy? First, let’s take a look at the example I used previously:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/76209911-fdb3-4ee9-a116-2cef11c821d1/final.png)

Sure, we could go with a similar structure, but we would be missing a key factor: location. Now, could we solve this with user based assignment? We could, but that would introduce some new challenges:

-   User assigned apps and policies are provisioned later in the Autopilot process, in most cases after the user has reached the desktop so there would be some lag in having the production ready build.
    
-   Magic Coffee currently does not have an up-to-date grouping structure for their users that reflects their location
    
-   PC inventory is managed per-site. So if an employee were to relocate to another location, they would be issued a new PC from that site. If an employee leaves, IT needs to re-purpose a device for a new user with the same, location based resources.
    
-   When using Autopilot pre-provisioning (formally known as “White Glove”) user assigned apps and policies do not apply.
    

So knowing this, what does our Group Tag structure need to look like? Well, if you recall from the last post, we used this schema:

**WIN-AP-CORP**

**_{PLATFORM}-{ENROLLMENT}-{BUILD}_**

Let’s go ahead and add a place for the site location. I’ll use **WIN-AP-NY-CORP**. But why did I put the location in _that_ particular spot? Check out my new diagram that I made specifically for Magic Coffee:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/855795c3-42b6-414a-a609-7efb871187e1/All.png)

There’s a lot going on here. The main difference from my original diagram is the horizontal movement. Those are the sites. My “All Autopilot” PCs group is still intact, and it contains all the site locations within it. And then within the site groups, are the build groups.

But wait a minute- why wouldn’t I do the _build_ first followed by the location? Easy; that’s not what the business needed. To highlight this, let’s zoom in on the NY location:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/dc8f7977-b043-4f30-ba9d-b55916698bf6/NY.png)

Remember, each site is run by it’s own IT department, each with their own standards. Here, the IT department can determine which standards apply to all builds, like Office 365 apps, Windows Update rings and security policy. Now within each build, network drives, printers, VPN connection and applications can be used, still all keeping with the NY site standards.

What happens in NY…
-------------------

Let’s zoom out one more time:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/4c515464-2938-4ecd-92fe-bac88e0395c2/zoom+out.png)

You can see, that everything that is assigned and scoped to the NY site does not affect additional sites, like Chicago. They will not inherit anything from NY with this structure, and are free to have their own versions of Office, Windows updates, etc. And if Magic Coffee ever does need to deploy something to all devices in the business, we still have the **WIN-AP**, “All Autopilot” device group which will catch every PC in every site.

Note that while only NY has the KIOSK build group, we have plenty of room to add it to the Chicago site if and when the time comes. Remember; needs of the business define the technical requirements.

So that takes care of Magic Coffee Co and their Autopilot Group Tag structure. But what about that other company, Global Operations, Inc.? Let’s save that for next time.
