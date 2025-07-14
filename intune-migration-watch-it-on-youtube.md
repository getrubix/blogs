---
author: steve@getrubix.com
date: Mon, 11 Sep 2023 14:18:08 +0000
description: '"If you’re wondering why I haven’t been writing much on here, it’s because
  I’ve been busy over on the Getrubix YouTube channel. Here is a quick summary of
  each Intune tenant to tenant device migration video so you can easily jump to an
  area of interest.New videos"'
slug: intune-migration-watch-it-on-youtube
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/intune-migration-watch-it-on-youtube_thumbnail.jpg
title: Intune Migration watch it on YouTube
---

If you’re wondering why I haven’t been writing much on here, it’s because I’ve been busy over on the [Getrubix YouTube channel](https://www.youtube.com/@getrubix9986/featured). Here is a quick summary of each Intune tenant to tenant device migration video so you can easily jump to an area of interest.

New videos are being added almost daily, so I’ll continue to update this list as it grows.

Part 1: Live demo
------------------

This is the first upload about our Intune migration process. I show a completely live the demo of the finished process and end-user experience.

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/IxSi6UGOikg?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

Part 2: Starting to build
-------------------------

In Part 2, we start constructing the actual StartMigrate.ps1 PowerShell script together, which is the foundation for the solution.

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/OTVt9z8eWkc?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

Part 3: Migrate user data
-------------------------

Part 3 is arguably the most important (and definitely the longest). I break down the steps we use to migrate the user data between profiles.

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/5g-7zkNVdMM?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

Part 4: Cleaning up
-------------------

This was right after my trip from Hershey Park, hence the chocolate. Part 4 is about removing Tenant A MDM enrollment details from the PC.

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/64JlHFmVyIM?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

Part 5: Leaving Tenant A
------------------------

Part 5 shows us how to un-join the PC from Tenant A, whether it’s Azure AD or Hybrid joined.

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/oOc3k1B0hUQ?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

Part 6: The bulk primary refresh token
--------------------------------------

Part 6 is all about the building the provisioning package (PPKG) and how to obtain the bulk primary refresh token.

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/hkWC1aTOxgQ?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

Part 7: Wrapping up the start
-----------------------------

In Part 7, we wrap up completing the StartMigrate.ps1 PowerShell script.

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/I2lA5VDxUDc?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

Part 8: Starting the post-migration tasks
-----------------------------------------

Part 8 starts our journey through the post-migration tasks and how we XML and PowerShell files to make it happen.

<div class="iframe-wrapper">
  <iframe src="https://www.youtube.com/embed/gIq8uO4jLMQ?feature=oembed" title="YouTube video player" frameborder="0" allowfullscreen></iframe>
</div>

It’s beyond awesome how many of you are checking out (and hopefully trying) our migration solution. If you missed it, here is the beginning of the series:

[Tenant to Tenant Intune Device Migration: The Beginning of a Series](https://www.getrubix.com/blog/tenant-to-tenant-intune-device-migration-the-beginning-of-a-series)

And here is the link to the GitHub repo which contains all the files needed:

[GitHub: IntuneMigration](https://github.com/stevecapacity/IntuneMigration)
