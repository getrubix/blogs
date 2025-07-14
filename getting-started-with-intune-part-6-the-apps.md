---
author: steve@getrubix.com
categories:
- intune
- security
- powershell
- automation
date: Fri, 08 Nov 2024 14:03:53 +0000
description: "If you’re just tuning in (no pun intended…I think), we’ve gone from evaluating environments to migrating policies and blending your current fleet with new, Autopilot devices. Now it’s time for Part 6: the thrilling, aggravating, and somewhat nerve-wracking world of app deployment. This is both straight forward and all over the place at the same time, so I’ll go through this in some steps, and hopefully stay on track."
slug: getting-started-with-intune-part-6-the-apps
tags:
- compliance
- automate
- security
- script
- intune
- automation
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/getting-started-with-intune-part-6-the-apps_thumbnail.jpg
title: Getting Started with Intune Part 6 The Apps
---

If you’re just tuning in (no pun intended…I think), we’ve gone from evaluating environments to migrating policies and blending your current fleet with new, Autopilot devices. Now it’s time for Part 6: the thrilling, aggravating, and somewhat nerve-wracking world of app deployment.

This is both straight forward and all over the place at the same time, so I’ll go through this in some steps, and hopefully stay on track.

Remember, there is a full companion video series to these blogs in the GetRubix YouTube [member channel](https://www.youtube.com/playlist?list=UUMOF6q8UjlE5AFO52ht-G_L6A).

Understanding App Personas: Matching Apps to Users and Devices
--------------------------------------------------------------

One of the first steps in managing applications effectively is understanding **app personas**. This means knowing who needs what to do their job and ensuring the right apps reach the right devices. What I find to be a common issue is that most organizations do not have this mapped out. But if you read [Part 3](https://www.getrubix.com/blog/getting-started-with-intune-part-3-the-phases) then you should be well on your way to having a strong design architecture and this is a big part of it.

Core Apps vs. Role-Specific Apps
--------------------------------

-   **Core Apps**: These are the must-haves that everyone across the board should have—think Office Suite, VPN clients, or endpoint security software. Deploying these to every device, regardless of the use-case keeps the baseline functionality uniform.
    
-   **Role-Based Apps**: Different job roles often require different sets of tools. Your design team might need Adobe Creative Suite, while finance needs their specialized accounting software. Mapping out these roles helps you create more targeted app deployment policies.
    
-   **Specialized Apps**: These are niche tools that only a select group needs. Whether it’s development software or industry-specific applications, these can be rolled out on an as-needed basis.
    

Required vs. Available Apps: When to Push and When to Let Users Choose
----------------------------------------------------------------------

Next, let’s talk about deciding between making apps **required** or **available**.

### Required Apps: The Essentials

Some apps are just non-negotiable. These are pushed directly to users’ devices to ensure they have what they need, when they need it, without intervention. Required apps might include:

-   **Security Software**: Anything related to device security or compliance should be deployed as required. You don’t want your users opting out of endpoint protection, after all.
    
    -   Commons apps:
        
        -   CrowdStrike
            
        -   CarbonBlack
            
        -   BeyondTrust
            
        -   SentinalOne
            
        -   TeamViewer
            

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ece16ad7-e321-41bf-a836-82feab6b636b/DALL%C2%B7E+2024-11-08+08.56.38+-+Create+an+animated-style+image+depicting+application+icons+playing+billiards.+Represent+the+icons+as+characters+gathered+around+a+pool+table%2C+each+wit.png)

-   **Productivity Tools**: Core productivity apps like the Microsoft Office Suite, which are essential for daily tasks, fall into this category.
    
    -   Common apps:
        
        -   Microsoft 365 Apps (Office)
            
        -   G Suite (Google Docs)
            
        -   Snaggit
            
        -   Zoom
            
        -   Slack
            
        -   Webex
            
        -   Google Chrome
            

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/b600d1e5-0f51-46f2-bada-f957d3a0632b/DALL%C2%B7E+2024-11-08+08.52.14+-+Create+a+hand-drawn+animation+style+image+of+application+icons+sitting+in+a+McDonald%27s+restaurant%2C+eating+together.+Represent+each+app+as+a+square+ico.png)

**The Benefit**: You’re ensuring that critical applications are always installed and up to date.

### Available Apps: Empowering User Choice

Available apps show up in the **Company Portal** (Intune’s self-service app store), letting users install them as needed. This is perfect for apps that are useful but not mission-critical. Examples include:

-   **Niche Productivity Tools**: Apps that only a portion of the company needs, like Visio or specialized document readers.
    
    -   Common apps:
        
        -   Visio
            
        -   Project
            
        -   Visual Studio
            
        -   Adobe Acrobat
            

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1c7de816-886e-40db-a291-00f209c4d5d7/DALL%C2%B7E+2024-11-08+08.53.51+-+Create+a+manga+comic-style+image+depicting+application+icons+practicing+karate+in+a+traditional+dojo+setting.+Each+app+should+be+represented+as+a+squa.png)

-   **Optional Tools**: Apps that enhance productivity but aren’t required by everyone, like certain note-taking tools or communication platforms.
    
    -   Common apps:
        
        -   Notepad++
            
        -   VLC
            
        -   7Zip
            

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/9d39fb0a-903b-4356-a3dd-2d82bea379a5/DALL%C2%B7E+2024-11-08+08.54.53+-+Create+a+realistic+image+of+application+icons+depicted+as+characters+hanging+out+on+a+street+corner.+Represent+Notepad%2B%2B%2C+VLC%2C+and+7-Zip+as+characters.png)

**The Benefit**: Users have the flexibility to install what they need without IT overloading their devices.

Different Methods to Deploy Apps Through Intune
-----------------------------------------------

We’ve covered app deployment quite a bit on the channel, so you know there are _many_ ways to do this. But as part of design, you should at least start thinking about which apps make sense to deploy with which method.

### 1\. **Win32 App Deployment**

-   **When to Use**: This is ideal for complex apps or apps that require custom installation scripts, such as .exe installers or apps with unique configuration requirements.
    
-   **Pros**: Full control over installation parameters, supports detection rules, and provides comprehensive installation monitoring.
    
-   **Cons**: More effort upfront to package and maintain.
    

### 2\. **Microsoft Store for Business (Legacy)**

-   **When to Use**: Don’t
    
-   **Pros**: None
    
-   **Cons**: Not applicable
    

### 3\. **Microsoft Store App (New)**

-   **When to Use**: For free-to-use and community apps that you want to deploy with minimal effort.
    
-   **Pros**: They don’t require any packaging as the apps are sourced from either the Microsoft Store or Winget.
    
-   **Cons**: For packages coming from Winget, you will need to do your due diligence to verify the integrity of the app due to it being a community tool with minimal security measures.
    

### 4\. **Line-of-Business (LOB) Apps**

-   **When to Use**: For custom-developed apps or specific line-of-business applications that need to be deployed as .msi files.
    
-   **Pros**: Simple for straightforward .msi-based apps.
    
-   **Cons**: No control over requirements or detection. Can conflict when deploying along-side **Win32 apps**
    

### 5\. **Web Apps**

-   **When to Use**: For apps that run primarily through a browser but need quick access from the device.
    
-   **Pros**: Fast setup and minimal configuration required.
    
-   **Cons**: Limited to browser-based functionality; not ideal for offline use.
    

Patch Management: Keeping Apps Up to Date Without Surprises
-----------------------------------------------------------

Deploying apps is one thing—keeping them up to date is where things get tricky. Here’s how to handle app patching effectively:

### Version Control and Updates

-   **Plan for Staged Rollouts**: Just like with Windows updates, consider a phased approach for app updates. Start with a small, controlled group before rolling out company-wide.
    
-   **Monitor for Issues**: Use Intune’s built-in monitoring tools to keep an eye on deployments and quickly identify any installation failures or version conflicts.
    
-   **Maintain Older Versions**: Always have a rollback plan. Keeping older versions of apps on standby can save the day when an update turns out to be more of a problem than a solution.
    

It’s important to note that Intune does not have an automated patching system for 3rd party apps. So, if you deploy and manage a large catalog of applications, it is definitely worth looking into a platform specifically designed for automated app management.

[Check out my video](https://youtu.be/oihMYHeaCQE) on the [ZeroTouch.AI](http://zerotouch.ai/) platform to learn about app automation.

### Communicating Updates

Keep users in the loop by notifying them about upcoming updates. This cuts down on the surprise factor and lets them know you’re making improvements (not just randomly changing things for fun).

Final Thoughts: Apps as the Heart of User Productivity
------------------------------------------------------

Managing applications in Intune means balancing user needs, security requirements, and IT sanity. By mapping out app personas, deciding between required and available apps, choosing the right deployment method, and handling patch management with care, you set your organization up for smoother operations and fewer 3 a.m. calls (if you’re the type that keeps your phone on).

The good news? As long as you go through proper design and plan things out, you can keep things easy so that IT can spend time on more important issues… like managing Windows and driver updates. Yeah- we’ll talk about that next.
