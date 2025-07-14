---
author: steve@getrubix.com
categories:
- intune
- security
- powershell
date: Fri, 01 Dec 2023 14:02:47 +0000
description: "After being inspired by Andrew Taylor’s weekly Intune newsletter (which now includes me for some reason), I figured it would make sense to start providing a summary of my videos every Friday. This will give me a chance to add some context, helpful links, and address some questions that may have come up throughout the week."
slug: weekly-recap-december-1
tags:
- intune
- powershell
- compliance
- script
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/weekly-recap-december-1_thumbnail.jpg
title: Weekly Recap December 1
---

After being inspired by [Andrew Taylor’s weekly Intune newsletter](https://andrewstaylor.com/2023/12/01/intune-newsletter-1st-december-2023/) (which now includes me for some reason), I figured it would make sense to start providing a summary of my videos every Friday. This will give me a chance to add some context, helpful links, and address some questions that may have come up throughout the week.

## Moving an Intune Tenant
---

My team has been long time users of [Mikael Karlsson’s](https://www.linkedin.com/in/mikael-karlsson-66154326/) Intune Management PowerShell tool, which you can grab [here from GitHub](https://github.com/Micke-K/IntuneManagement). It’s incredibly useful for documenting your Intune tenant.

Now that we’re (obviously) in the weeds of migrating devices with Intune, it makes sense to talk about migrating the _actual_ Intune tenant, and this management tool does an amazing job of exporting configs from one environment and importing into another.

### Watch the YouTube video here:

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/b15-Vy8TS6E?start=2feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

Intune Custom Compliance
------------------------

The ability to create custom compliance policies within Intune is one of the best kept secrets of the platform.

It can be a bit tricky to get started between the two types of scripts that are used, but I posted the samples I used in the video [here in the GitHub](https://github.com/stevecapacity/IntunePowershell) to make things easier.

### Watch the YouTube video here:

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/uyu-TZBtE2M?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

Intune Device Migration V3
--------------------------

The original user experience was the first thing I showed a few months ago. Since then there have been so many new members to the community that it made sense to record an updated walkthrough.

I will be releasing the V3 code on GitHub by mid-next week, but here is a snippet from the README.md in advance:

> Intune tenant-to-tenant device migration V3 adds the capability of operating off of one, settings file that is modified so that the solution code can remain untouched.

The migration solution offers two paths:
 
1. **User data migration**: To migrate user data during migration, start by deploying the _PreMigrate_ script prior to running the full migration package
2. **No user data migration**: If user data does not need to be migrated (perhaps you're utilizing OneDrive for that), you can start the process with the migration package at the start _StartMigrate_ script. This can automatically detect if the _PreMigrate_ script has been run and proceed accordingly.

### Watch the YouTube video here:

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/LiHj0k5WExQ?start=241feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

### Thank you

That does it for this week. I want to thank the entire GetRubix community, as we’re now over **80 users** in the [official Discord server](https://discord.gg/getrubix). If you’re not already in there, feel free to stop by and say hi.
