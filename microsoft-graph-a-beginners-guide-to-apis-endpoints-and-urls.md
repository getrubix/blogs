---
title: "Microsoft Graph A Beginners Guide to APIs Endpoints and URLs"
slug: "/blog/microsoft-graph-a-beginners-guide-to-apis-endpoints-and-urls"
date: "Sun, 08 Dec 2024 01:55:14 +0000"
author: "stevew1015@gmail.com"
description: ' This week, we kicked off the GetRubix YouTube series, Getting Started with Graph, where we explore… well… getting started with using the Microsoft Graph.One piece of feedback I’ve received pretty consistently is that you all seem to love the graphic breakdown of Graph calls and queries. The second'
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/microsoft-graph-a-beginners-guide-to-apis-endpoints-and-urls_thumbnail.jpg
---

This week, we kicked off the **GetRubix YouTube series, [Getting Started with Graph,](https://www.youtube.com/playlist?list=PLKROqDcmQsFls8cPHk3HFz2mUURHx46_O) where we explore… well… getting started with using the Microsoft Graph.

One piece of feedback I’ve received pretty consistently is that you all seem to love the graphic breakdown of Graph calls and queries. The second piece of feedback? It’s a LOT to understand and can feel overwhelming—especially when trying to figure out which endpoints represent which areas of the M365 context.

To make things a bit easier, this post will serve as a reference guide on how to construct Graph calls so you can easily revisit it throughout your Graph adventures.

Anatomy of the URL
------------------

The Microsoft Graph URL looks similar to the address for most websites. For example:

-   **URL for all Intune-managed devices:**  
    `https://graph.microsoft.com/beta/deviceManagement/managedDevices`
    
-   **URL for all users in Entra:**  
    `https://graph.microsoft.com/v1.0/users`
    

Even though it might not seem like it at first, there’s a method to the madness of these URL structures. Let’s use the Intune example for a breakdown:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e29597be-45da-45d8-8292-9279dd0b8cd8/Graph+Queries.png)

From what I understand, many of you were screenshotting this from the videos anyway…

Just like in my [Part 2 video](https://youtu.be/X7IsmFldVAU), I’ve broken down and labeled each component of the URL:

1.  **API Address:**  
    This is the primary part of the URL, indicating which API we’re accessing. In this case, it’s Microsoft Graph… obviously.
    
2.  **Version:**  
    You might have noticed that some URLs have `/beta` in them while others have `/v1.0`. That’s because Microsoft publishes different versions of APIs:
    
    -   `**/v1.0**`: Generally available (GA) and the most stable version.
        
    -   `**/beta**`: May contain potential instabilities but is my personal preference because it offers more capabilities. Fun fact: Microsoft often uses `/beta` within their own consoles for many features.
        
3.  **Endpoint:**  
    This specifies the area of the API you’re accessing. In this case, it’s Intune (also referred to as `deviceManagement`).
    
4.  **Data Set:**  
    This represents the specific "thing" you’re working with, like devices, apps, policies, or updates. For example, `/managedDevices` in the URL is equivalent to selecting “Devices” inside the Intune console.
    

Common Endpoints and Data Sets
------------------------------

So, how do you put all of this together to navigate the Graph effectively?

Good news: now that we understand how the URL is structured, navigation becomes much simpler. I recommend using the `/beta` version as a starting point for most of your work. This means all your Graph URLs will begin with:  
`https://graph.microsoft.com/beta`

> **METHODS:** Methods are the actions we use to interact with endpoints, such as **GET, PATCH, POST,** and **DELETE.** While we’ll dive deeper into these in future posts, a great starting point is a simple **GET** to fetch data.

Here are some key areas you should become familiar with when working with Graph:

### **Intune**

-   **Managed devices:**  
    `https://graph.microsoft.com/beta/deviceManagement/managedDevices`
    
-   **Applications:**  
    `https://graph.microsoft.com/beta/deviceAppManagement/mobileApps`
    
-   **Configuration profiles:**  
    `https://graph.microsoft.com/beta/deviceManagement/configurationPolicies`
    
-   **Mobile app protection policies:**  
    `https://graph.microsoft.com/beta/deviceAppManagement/managedAppPolicies`
    

### **Entra**

-   **Devices:**  
    `https://graph.microsoft.com/beta/devices`
    
-   **Users:**  
    `https://graph.microsoft.com/beta/users`
    
-   **Enterprise applications:**  
    `https://graph.microsoft.com/beta/applications`
    
-   **Conditional Access policies:**  
    `https://graph.microsoft.com/beta/identity/conditionalAccess/policies`
    

### **Autopilot**

-   **Registered devices:**  
    `https://graph.microsoft.com/beta/deviceManagement/windowsAutopilotDeviceIdentities`
    
-   **Enrollment profiles:**  
    `https://graph.microsoft.com/beta/deviceManagement/windowsAutopilotDeploymentProfiles`
    

One Step at a Time
------------------

If you’ve recently stepped into Intune management and now find yourself trying to figure out APIs, it’s completely understandable to feel overwhelmed. That’s exactly why we’re doing this series.

The key is to take it one step at a time. Today, we focused on understanding endpoints. Next, we’ll tackle methods.

And if you’re thinking, “I can’t do this—I’m not a developer or a programmer,” just remember: neither am I.
