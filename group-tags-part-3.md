---
author: steve@getrubix.com
categories:
- powershell
- automation
date: Tue, 03 Dec 2019 15:59:00 +0000
description: >
  Sometimes the best and worst thing you can do with your work is show it to someone else. I knew something was missing from my first two scripts to automate the addition of group tags during Autopilot enrollment. If you haven’t yet, read the first and second posts in the series, or this will make zero sense from here on out.
slug: group-tags-part-3
tags:
- powershell
- script
- automate
- flow
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/group-tags-part-3_thumbnail.jpg
title: Group Tags Part 3
---

Sometimes the best and worst thing you can do with your work is show it to someone else. I knew something was missing from my first two scripts to automate the addition of group tags during Autopilot enrollment. If you haven’t yet, read the [first](https://z0touch.home.blog/2019/11/30/autopilot-group-tags/) and [second](https://z0touch.home.blog/2019/12/02/group-tags-live/) posts in the series, or this will make zero sense from here on out.

I brought my PowerShell scripts to my right-hand engineer on my team, Jesse, who immediately found and corrected some glaring flaws. After cursing at him for seeing in two minutes what I couldn’t see after hours of staring at my laptop, I rejoiced at the new pieces Jesse was able to add. Not only did we clean up the null values showing in the pop-up window, but the ability to add a new group tag on the spot was created. And because he didn’t like that the previous posts were missing a “Part 1/2” moniker, I added Part 3 here to make him happy.

No more nulls
-------------

My second iteration of the group tag selector script was able to scrape all existing group tags for Autopilot devices from the Microsoft Graph. You can download that version of the [script here](https://z0tinstallers.blob.core.windows.net/configs/tagSelector2.ps1?sv=2019-02-02&ss=b&srt=sco&sp=rwdlac&se=2020-12-31T22:44:02Z&st=2019-12-02T14:44:02Z&spr=https&sig=kg2j7kOXkWNZQieiYVhDoqVfIzCA1npLF7eYtoTwQ4I%3D). The problem was I couldn’t seem to filter out the null values that represented ’empty’ group tags.

![Let’s be honest; that’s disgusting](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090890315-0EZUWB9IQCY4ULIVGR2M/image-asset.png)

Let’s be honest; that’s disgusting

It wasn’t for lack of trying. I played with various _if_ statements to try and leave those blank values out. But of course, Jesse cracked this one right away. Here was the original code block that listed the values from the XML:

![image.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090919355-WIZB3PS14854LTAAHANU/image.png)

The **$tag** variable is just listing each group tag in the XML, regardless of whether or not it was empty. In the end, it was a simple _if_ statement inside the _foreach_ loop, excluding any entries equal to nothing.

![image-1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090945897-XFN1PSOYPORV6826ACM8/image-1.png)

So, that took care of leaving out the null values.

New tag, no tag, and cancel
---------------------------

Of course, the first question Jesse asked me after reviewing the script was “where can I add a new group tag?”… to which I replied “shut up”. It was on my radar of things to come, but hadn’t worked out the logistics of putting that into the list box form and dealing with the dialog pop-up. But again, Jesse cooked it up pretty quickly, along with an option to not have a tag, or to flat-out cancel. Here is the workflow for the logic:

-   **$tag** variable is set to the **selected item** of the list box form.
    
-   If the **selected item** is one of the displayed tags, it simply becomes the tag (just as it was before).
    
-   Two new entries were added to the list box in addition to the dynamic listing of tags from the graph: **_“No tag”_** and **_“Add new tag…”_**
    

![image-3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581090998998-91WXPKSH4B6TEKZSDHNP/image-3.png)

-   If **_“No tag”_** is the selected item, the **$tag** variable is simply set to **$null** and the device is enrolled with no tag; perfect!
    

![image-4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581091034056-IZUY8GPNBB4XQ0381F0R/image-4.png)

-   If **_“Add new tag…”_** is the selected item, another form is displayed as a text box, where a brand new tag can be entered. The **$tag** variable is then set to that string.
    

![image-5.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581091055091-XQBBPTY64PLMHDW7MB56/image-5.png)

Also, proper handlers for the “cancel” button were also added. Now when the script is run, this is the pop-up window displayed:

![Notice the ‘Add new tag…’ option](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581091133882-R9NCGOTBVRWQE9ITMCWR/2019-12-03-09_00_36-ztds-1903-1203-on-sweiner-14178-virtual-machine-connection.png)

Notice the ‘Add new tag…’ option

When **“Add new tag…”** is selected, a new text box appears:

![2019-12-03-09_01_14-ztds-1903-1203-on-sweiner-14178-virtual-machine-connection.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581091165997-PR5FWNRTN9N65B9MRQD4/2019-12-03-09_01_14-ztds-1903-1203-on-sweiner-14178-virtual-machine-connection.png)

There are probably a few things that can still be cleaned up, but I think that gets us where we want to be. Again, thanks to Jesse for taking this to the next level. You can grab the new version of the [script here](https://z0tinstallers.blob.core.windows.net/configs/tagSelector3.ps1?sv=2019-02-02&ss=b&srt=sco&sp=rwdlac&se=2020-12-31T22:44:02Z&st=2019-12-02T14:44:02Z&spr=https&sig=kg2j7kOXkWNZQieiYVhDoqVfIzCA1npLF7eYtoTwQ4I%3D).

Let me know how it works for you!
