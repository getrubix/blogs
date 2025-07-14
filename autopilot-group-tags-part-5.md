---
author: steve@getrubix.com
date: Wed, 16 Mar 2022 14:58:21 +0000
description: '"Alright- we may be at the end here. In Part 2 I mentioned we were going
  to be looking at a Group Tag structure for two example companies: Magic Coffee Co
  and Global Operations Inc. I’m fairly certain they’re both imaginary.We spent time
  with Magic Coffee"'
slug: autopilot-group-tags-part-5
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-group-tags-part-5_thumbnail.jpg
title: Autopilot Group Tags Part 5
---

Alright- we may be at the end here. In [Part 2](https://www.getrubix.com/blog/autopilot-group-tags-part-2) I mentioned we were going to be looking at a Group Tag structure for two example companies: Magic Coffee Co and Global Operations Inc. I’m fairly certain they’re both imaginary.

We spent time with Magic Coffee building a Group Tag structure to fit their need to separate device builds based on site locations. As a recap, here is was designed for them:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7ca33ee2-27ca-46df-9cad-8c9caae281db/All.png)

This structure reflects the need to identify each device build per site, leaving room for specific app assignments in addition to future builds. But take a closer look; the primary differentiator here is the location, which makes sense because Magic Coffee allows the IT admins per site to control the build requirements. But this is very different for our other business; enter Global Operations Inc.

Going global
------------

It may shock you, but Global Operations Inc. (or “Global Opps” for short), is a 75,000 user organization that operates globally. They have locations in 7 major regions and various builds and use cases within them. However, unlike Magic Coffee, Global Opps wants to implement a build standard for their devices across all regions. Here is the breakdown:

Locations:

-   US
    
-   Canada
    
-   Mexico
    
-   Sweden
    
-   UK
    
-   Germany
    
-   India
    

Device Builds:

-   Corporate
    
-   Call Center
    
-   Point-of-Sale
    

There are a lot of factors here. But again, it’s all about requirements and the needs of the business. Let’s start with the global standard. Global Opps wants to make sure the corporate build is consistent whether it’s deployed in Canada or India. That means the build should be my primary differentiator with this Group Tag structure. So let’s start there:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/07813967-f244-46d7-93d9-2b05d46f6414/Screen+Shot+2022-03-15+at+3.08.05+PM.png)

Just like the rest of the examples, I have my parent group, **WIN-AP**, that is collecting all tagged devices. Next are the device builds. They come before any location tag because of the need for the standard. Core applications and policies are applied here so that the build components cascade through subsequent tags.

While Global Opps is enforcing a global standard for these builds, they will also allow IT admins in each region to have control over additional apps or policies for their region only. This can be done by applying [RBAC](https://docs.microsoft.com/en-us/mem/intune/fundamentals/role-based-access-control) (Role-based-access-control) scope tags to our dynamic groups. I’ll get into RBAC scope tags in a future post, as they ARE NOT the same as group tags- how many more times can I type “tags” in this paragraph?

Here is the structure to support the regions:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/9ca73a09-38dc-43ce-9780-33dd6a67e7e8/Screen+Shot+2022-03-15+at+3.44.45+PM.png)

Each build as subsequent tags for the region. Central IT will determine what is configured at the **WIN-AP-CORP** level. However, the IT admins in Canada have the ability to manage **WIN-AP-CORP-CA** the way they see fit for their needs. They can do this without interfering with the corporate standard.

Wrapping up the tags
--------------------

To close out the 5-part series, I want to stress how important it is to leverage Group Tags in your environment. It’s without a doubt, the most effective way to organize Autopilot devices. Having a structure will also help you provide a consistent provisioning experience because you’ll know exactly which components are deployed at each step in the process.

As always, feel free to reach out and let me know if you have any questions about any of this. Have fun deploying!
