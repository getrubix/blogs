---
author: steve@getrubix.com
categories:
- intune
- azure
date: Sun, 27 Oct 2024 13:40:03 +0000
description: "Welcome to my series on adopting Microsoft Intune in your organization. This series isn't exclusively technical; it’s more of a practical, real-world approach to help you get started while considering key factors. Additionally, I’ll have a video guide to complement these articles on the GetRubix YouTube members' channel."
slug: getting-started-with-intune-part-1-where-are-you-today
tags:
- active directory
- intune
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/getting-started-with-intune-part-1-where-are-you-today_thumbnail.jpg
title: Getting Started with Intune Part 1 Where Are You Today
---

Welcome to my series on adopting Microsoft Intune in your organization. This series isn't exclusively technical; it’s more of a practical, real-world approach to help you get started while considering key factors. Additionally, I’ll have a video guide to complement these articles on the [GetRubix YouTube members' channel](https://www.youtube.com/channel/UCF6q8UjlE5AFO52ht-G_L6A/join).

Let me guess
------------

When you deploy new PCs to end users, you probably load a custom image, join them to your Active Directory domain, and manage them using a combination of Group Policy Objects (GPOs), SCCM (also known as Configuration Manager or MECM), or a third-party tool like Manage Engine, Ivanti, or KACE. You might have a VPN that allows users to access on-premises resources remotely, while also relying on it for domain connectivity and policy updates.

For mobile devices, such as iOS and Android, you may use Airwatch (Workspace ONE), MobileIron, or MaaS360. You might also allow users to use their own mobile devices for corporate email.

Maybe you’re using JAMF or Kandji to manage Macs, or perhaps they’re just floating around unmanaged.

Then, the higher-ups tell you: it’s time to consolidate everything into Microsoft Intune.

Does any of this sound familiar?

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/b4109908-0c9d-4fc1-ac70-8d0339f0a00c/fortuneTeller.jpeg)

My last 8 years
---------------

No, I’m not reading your mind. But for almost eight years, I’ve had this same conversation with many organizations—over 1,500, to be exact. A recent review with my team revealed that since January 2017, we’ve implemented Intune for device management in over 1,500 organizations.

I’m not sharing this to brag but to convey just how deeply I’ve been involved in the world of modern management. While there have been challenging moments, the hardest obstacles are rarely technical.

The human element
-----------------

Changing deployment methods, management tools, and operational practices isn’t easy. The larger the organization, the harder it can get. Can we take your GPO settings and convert them into Intune policies? Absolutely. But restructuring how those policies are managed and rolled out, or possibly even switching the resource responsible for them? That’s a different story.

For every technical challenge, you’ll encounter twice as many process or operational hurdles.

Fortunately, despite each organization's unique needs, there are foundational components that rarely differ among them. To make headway into the Intune world, start by understanding your current device management landscape.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/23e028c6-e6b9-4c9b-a3cb-92395303a341/human+%281%29.jpeg)

What do you do today?
---------------------

Before moving forward, it’s essential to know where you are today. Depending on the type of organization you manage, this might be complex. Here are some questions to help you map out your current environment:

**For PCs:**

-   What is your onboarding and deployment process?
    
-   How are machines imaged?
    
-   How many build types do you have?  
    (e.g., corporate laptops, manufacturing desktops, front-line kiosks, shared training desktops)
    
-   How many OEMs do you deploy?  
    (e.g., Dell, Lenovo, HP, Microsoft Surface)
    
-   How are PCs patched?
    
-   Do you use certificate infrastructure?  
    (Root CA, PKI, SCEP)
    
-   What tools do you use for network access?
    
-   What GPOs are deployed to PCs?
    
-   What tools do you use for various PC management tasks, including:
    
    -   Remote management
        
    -   Monitoring
        
    -   Software and hardware inventory
        
-   Are Macs used in the organization? If so, gather similar data for them.
    

**For Mobile:**

-   Is Bring Your Own Device (BYOD) allowed?
    
-   Do you register devices in enrollment programs?  
    (Apple Business Manager \[ADE, formerly DEP\], Samsung KNOX, Android Enterprise)
    
-   What MDM (Mobile Device Management) solution is in place?
    
-   How is corporate data secured on mobile devices?
    
-   Are devices shared among users?
    
-   What types of applications are used?  
    (Store apps, custom in-house applications)
    
-   Do mobile devices access internal resources?
    

**User Experience:**

-   Do users work remotely, in-office, or in a hybrid setup?
    
-   Are applications used by different departments cataloged?
    
-   How do users request new software?
    
-   How do users request support?
    
-   When a new user is onboarded, how long does it take for them to receive their hardware?
    

Somewhere to start
------------------

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/812df48e-a58b-4326-af17-05054d06df5c/running.jpeg)

I know—it’s a lot of information to gather. More important than knowing all these details is identifying who holds this knowledge. In most medium to large organizations, collecting this data requires teamwork, and that’s okay.

Transitioning to a consolidated and modern platform like Microsoft Intune is a transformational process that benefits from a "circle of excellence." This circle not only helps in gathering information but also facilitates discussions on changing responsibilities as Intune consolidates many management tasks into a unified platform.

In the next part, we’ll use the information gathered here to match it with Intune’s capabilities and begin building a design architecture.
