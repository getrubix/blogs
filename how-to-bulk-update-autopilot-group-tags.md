---
author: steve@getrubix.com
date: Sat, 06 Apr 2024 23:13:09 +0000
description: '"Well, I almost made it a full vacation without working. But hey- what
  can you do?The other day, I received a question in the Discord server about what
  the easiest way would be to update enrollment profiles for Autopilot devices, specifically
  based on the group tag."'
slug: how-to-bulk-update-autopilot-group-tags
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/how-to-bulk-update-autopilot-group-tags_thumbnail.jpg
title: How to Bulk Update Autopilot Group Tags
---

Well, I almost made it a full vacation without working. But hey- what can you do?

The other day, I received a question in the [Discord server](https://discord.gg/getrubix) about what the easiest way would be to update enrollment profiles for Autopilot devices, specifically based on the group tag. The obvious answer is “change the group tag”. But how do we do that in bulk?

I wrote a quick PowerShell script using the Graph API to change group tags in bulk.

We start by authenticating to the graph via the **WindowsAutopilotIntune** module.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/71d4259b-0a06-4e73-be21-5e72ee4ba385/Screenshot+2024-04-06+190336.png)

You’ll also need NuGet to install the module so we threw that in. Afterwards, use **Connect-MgGraph** to authenticate to your tenant (permissions will be required, of course.)

So now it’s time to re-tag. But how should we do this? I thought of two possible scenarios: the first, gathering a list of serial numbers for the devices that need to be updated. We can then import the list and use that to change tags.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e059a59b-cf8a-4a53-ac0e-390c85870869/Screenshot+2024-04-06+190814.png)

That works.

But let’s say instead of individual devices, you’re looking for everything that has a certain group tag? Let’s call that the “old tag”. We can use the old tag to target those devices for tag changing.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/64dabd7b-6e5d-4c7e-8cf8-e556d30c16dc/Screenshot+2024-04-06+190949.png)

There we go! The full code is now on my GitHub page [here](https://github.com/stevecapacity/IntunePowershell/blob/main/bulkGroupTagUpdate.ps1).

That’s all for now, but stay tuned for new videos coming this week, including _another_ MVP guest!
