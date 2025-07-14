---
author: steve@getrubix.com
date: Wed, 13 Nov 2024 21:34:16 +0000
description: '"We’ve been through a lot over the last few weeks. Cloud vs hybrid,
  group policy vs CSP, WSUS vs Update rings; it’s been a ride. But at this point,
  you should have all the fundamental tools you need get Intune off the ground. In
  this"'
slug: getting-started-with-intune-part-8-your-new-management-world
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/getting-started-with-intune-part-8-your-new-management-world_thumbnail.jpg
title: Getting Started with Intune Part 8 Your New Management World
---

We’ve been through a lot over the last few weeks. Cloud vs hybrid, group policy vs CSP, WSUS vs Update rings; it’s been a ride. But at this point, you should have all the fundamental tools you need get Intune off the ground. In this final part, let’s look at what your endpoint management life will look like day-to-day.

Remember, there is a full companion video series to these blogs in the GetRubix YouTube [member channel](https://www.youtube.com/playlist?list=UUMOF6q8UjlE5AFO52ht-G_L6A).

Daily Intune Operations: Rethinking the IT Routine
--------------------------------------------------

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/92048788-a8c9-4b43-bcc8-7ff342d57e68/DALL%C2%B7E+2024-11-13+16.01.01+-+A+vibrant+image+illustrating+the+theme+of+changing+a+routine.+A+person+is+depicted+transitioning+from+a+busy%2C+repetitive+schedule+shown+on+the+left+si.png)

In traditional Windows management, the daily routine might involve everything from managing group policies and pushing updates through WSUS to troubleshooting specific devices on-site. Intune flips much of this on its head, moving device management to the cloud, so you can support devices from anywhere. Here are some of the biggest differences:

### 1\. Policy Management – Set It and (Mostly) Forget It

With Intune, device and user policies are **cloud-based** and deployed automatically to enrolled devices. Once you’ve configured a policy, Intune takes care of applying it across all targeted devices.

-   **Less Hands-On Adjustment**: Unlike group policies, which often required manual intervention to troubleshoot, Intune policies are more of a “set it and forget it” setup. If something isn’t working, troubleshooting typically involves checking device compliance or the user’s enrollment status.
    
-   **Real-Time Adjustments**: You can push out policy changes instantly, and they’re applied the next time the device checks in. It’s a smoother process compared to group policy updates, which rely on network connections and might take several restarts to fully apply.
    

### 2\. Update Management – Streamlined with Update Rings

Instead of diving into WSUS for manual approvals and tracking, Intune’s **update rings** allow you to automate updates and monitor compliance through a single dashboard.

-   **Centralized Compliance Reporting**: Intune’s reporting tools make it easy to see at a glance which devices are up-to-date and which need a push. With compliance policies in place, you’ll know instantly if a device falls behind.
    
-   **Conditional Access Integration**: Devices that don’t meet update requirements can be restricted automatically through Conditional Access, so you’re spending less time manually tracking down non-compliant devices.
    

### 3\. Support and Troubleshooting – Remote Assistance on Steroids

With Intune, you’re not limited by on-premises restrictions, which means your team can troubleshoot remotely and support users wherever they are.

-   **Remote Actions**: From wiping a lost device to forcing a sync or resetting a passcode, Intune allows you to perform these actions remotely. It’s far less “hands-on” than traditional IT support, which relied heavily on physical access.
    
-   **Self-Service Capabilities**: The **Company Portal** app gives users a bit more independence with access to available apps, device check-in, and compliance checks. This can help reduce help desk calls for minor issues and gives users an easy way to handle some of their own IT needs.
    

Autopilot vs. Traditional Imaging: The End of “Ghosting” Machines
-----------------------------------------------------------------

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/cb826878-7f0e-4fb3-ab26-c162a5a0b31c/crying.png)

Now, let’s talk about deploying new PCs. If you’ve spent years using traditional imaging tools, you’re probably familiar with the endless cycle of capturing and deploying images, then customizing them for specific departments. **Windows Autopilot** takes a different approach, and it’s making life a lot easier (and faster) for IT.

### ZERO TOUCH

Windows Autopilot allows you to provision new devices straight from the manufacturer to the end-user, without ever having to image or reimage the machine. You configure the setup process in Intune, and Autopilot takes care of the rest. Here’s how it compares to traditional imaging:

-   **Zero-Touch Deployment**: With Autopilot, users receive their devices, power them on, connect to Wi-Fi, and sign in. The device then pulls down configurations, apps, and settings directly from Intune, customizing itself according to the user’s profile.
    
-   **No More Image Maintenance**: Traditional imaging requires maintaining, updating, and testing different images. With Autopilot, you only need to update the configuration profiles in Intune, so every deployment is based on the latest setup without any additional effort on your part.
    
-   **End-User Experience**: Users get a more personalized experience with Autopilot because the device is set up specifically for them, rather than a generic imaged setup that might require additional customization.
    

### Key Differences and Benefits

-   **Faster Deployment Times**: Forget the hour-plus imaging process. With Autopilot, deployments are streamlined, and users can be up and running in under 30 minutes.
    
-   **Device Lifecycle Management**: Devices can be reset and reconfigured for new users with Autopilot. This makes repurposing machines simpler, as each new user gets a fresh configuration without the need to reimage.
    
-   **Better Security**: Autopilot devices can be set up with strict compliance policies and Conditional Access right from the start, ensuring that they meet security standards immediately upon setup.
    

Day-to-Day Management with Intune: Staying Proactive
----------------------------------------------------

Managing Intune isn’t about reacting to problems; it’s about staying proactive. The cloud-based nature of Intune means you can work smarter with automated policies, reports, and alerts. Here are a few tips to keep things running smoothly:

-   **Regular Reporting**: Check Intune’s compliance and device health reports weekly. This helps you catch any issues early and address devices that may have fallen out of compliance.
    
-   **Monitor for New Features**: Microsoft frequently rolls out new capabilities in Intune, so keep an eye on updates. New features can often streamline processes, making it easier to adjust to ever-evolving organizational needs.
    
-   **Stay Flexible**: The biggest adjustment in managing Intune day-to-day is embracing the cloud’s flexibility. Changes and configurations are applied quickly, so don’t be afraid to test new policies or tweak settings as your organization’s needs evolve.
    

Final Thoughts: Wrapping Up the Intune Journey
----------------------------------------------

Moving from traditional Windows management to Intune is a big shift, but with Autopilot, update rings, and cloud-based management, your day-to-day IT routine can be more efficient than ever. It might take some time to adjust, but once you’re up and running, you’ll find that Intune is less about troubleshooting and more about proactive management.

I really hope the “less-technical” approach with this series was helpful, especially if you caught any of my commentary videos on it. There’s a lot more planned in the “Getting started” world so keep checking back.
