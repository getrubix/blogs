---
author: steve@getrubix.com
categories:
- intune
- powershell
date: Mon, 02 Dec 2019 15:51:00 +0000
description: "In the last post, we walked through a PowerShell script I created to enroll devices in Autopilot with a pop-up window to choose a group tag. This should be helpful for existing ‘one-off’ devices or testing different deployment profiles. Also, many of you asked me for the script itself."
slug: group-tags-live
tags:
- intune
- script
- powershell
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/group-tags-live_thumbnail.jpg
title: Group Tags LIVE
---

In [the last post](https://z0touch.home.blog/2019/11/30/autopilot-group-tags/), we walked through a PowerShell script I created to enroll devices in Autopilot with a pop-up window to choose a group tag. This should be helpful for existing ‘one-off’ devices or testing different deployment profiles. Also, many of you asked me for the script itself as I only posted pieces of it last time; my bad. You can download the [entire script here](https://z0tinstallers.blob.core.windows.net/configs/tagSelector.ps1?sv=2019-02-02&ss=b&srt=sco&sp=rwdlac&se=2020-12-31T22:44:02Z&st=2019-12-02T14:44:02Z&spr=https&sig=kg2j7kOXkWNZQieiYVhDoqVfIzCA1npLF7eYtoTwQ4I%3D).

The biggest issue with my first run at this is how we aggregate the actual group tags. Basically, I created an XML file listing the tags, and used that to populate the list box form in PowerShell (yes, I realize the incredible ‘_lameness’_ of this). The optimal way would be to pull in the group tags directly from the Microsoft Graph. Fortunately, after consuming too much coffee after 9 pm last night, success was achieved.

First, rearrange
----------------

Here is the chunk from the first draft of the script that downloaded the group tag config file.

![2019-12-02-09_58_56-tagselector.ps1-grouptags-visual-studio-code.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090372669-ZRAO0N3O1VWREAOCKQCV/2019-12-02-09_58_56-tagselector.ps1-grouptags-visual-studio-code.png)

So first things first; we’re no longer going to be getting the XML file from a static location. Instead, we’re going to make a modified graph call to return active group tags. Note that this will only work on currently applied group tags.

Here is the graph call:

**Get-AutopilotImportedDevice** **| select -ExpandProperty groupTag | Group-Object | % Name**

Now our code to get the group tags from an XML file looks like this:

![Because we’re still getting the tag content from an XML, we’re exporting our graph call as an XML file next to the script.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090405134-PU1EY3MENTNYNP4U6R0W/2019-12-02-10_34_25-tagform2.ps1-grouptags-visual-studio-code.png)

Because we’re still getting the tag content from an XML, we’re exporting our graph call as an XML file next to the script.

Next, move the modules
----------------------

Because we’re using the **Get-AutopilotImportedDevice** function, we have to install and import our modules and authenticate to the graph **BEFORE** getting the tags. This function specifically comes from the [WindowsAutopilotIntune module](https://www.powershellgallery.com/packages/WindowsAutoPilotIntune/3.9) by Michael Niehaus. In the previous script, this was done later.

### Before…

![We need to move this block up above](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090444159-D9JR2Y3PS014KFNMNMUE/2019-12-02-10_37_11-tagselector.ps1-grouptags-visual-studio-code-1.png)

We need to move this block up above

### After…

![2019-12-02-10_39_40-e2978f-tagform2.ps1-grouptags-visual-studio-code-2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090640841-N9W19DEJV4AOP0POX6KF/2019-12-02-10_39_40-e2978f-tagform2.ps1-grouptags-visual-studio-code-2.png)

Finally, enjoy!
---------------

Now we will have authenticated prior to getting the tags, since they’re coming straight from the graph. Here is the new pop up:

![2019-12-02-10_50_01-test-no-ap-1903-on-sweiner-14178-virtual-machine-connection.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090670314-4TC190B2NDOR4DENZT33/2019-12-02-10_50_01-test-no-ap-1903-on-sweiner-14178-virtual-machine-connection.png)

So we’re getting the same options as before, just not as clean looking. Ultimately, I still have to do some work on getting rid of the null values, but this is at least functional.

The entire new script can be [downloaded here](https://z0tinstallers.blob.core.windows.net/configs/tagSelector2.ps1?sv=2019-02-02&ss=b&srt=sco&sp=rwdlac&se=2020-12-31T22:44:02Z&st=2019-12-02T14:44:02Z&spr=https&sig=kg2j7kOXkWNZQieiYVhDoqVfIzCA1npLF7eYtoTwQ4I%3D). Let me know what you think.
