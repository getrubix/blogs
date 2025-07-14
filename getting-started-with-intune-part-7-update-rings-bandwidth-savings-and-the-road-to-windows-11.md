---
author: steve@getrubix.com
categories:
- intune
- security
- automation
- azure
date: Tue, 12 Nov 2024 21:54:05 +0000
description: "Policy has been configured. Personas are mapped out. Apps are packaged and ready for deployment. We’re so close to getting Intune off the ground and there’s only one, tiny part left… Windows updates. Depending on what you’re used to, this may be one of the biggest."
slug: getting-started-with-intune-part-7-update-rings-bandwidth-savings-and-the-road-to-windows-11
tags:
- compliance
- security
- entra
- intune
- automation
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/getting-started-with-intune-part-7-update-rings-bandwidth-savings-and-the-road-to-windows-11_thumbnail.jpg
title: Getting Started with Intune Part 7 Update Rings Bandwidth Savings and the Road
  to Windows 11
---

Policy has been configured. Personas are mapped out. Apps are packaged and ready for deployment. We’re so close to getting Intune off the ground and there’s only one, tiny part left… Windows updates.

Depending on what you’re used to, this may be one of the biggest shifts in endpoint management when you adopt Intune. I’ll try to cover all the bases and take you through the important stuff.

Remember, there is a full companion video series to these blogs in the GetRubix YouTube [member channel](https://www.youtube.com/playlist?list=UUMOF6q8UjlE5AFO52ht-G_L6A).

Why Move from WSUS to Windows Update for Business?
--------------------------------------------------

If you’ve been in IT for a while, you’re probably familiar with WSUS. It’s been the go-to for Windows updates, offering centralized control over what gets installed and when. But with more organizations moving to remote and cloud-based management, WSUS is starting to show its age. Enter Windows Update for Business, which provides a cloud-friendly, hands-off way to handle updates without requiring on-premises infrastructure.

Here’s why WUfB can make your life easier:

1.  **Flexibility for Hybrid and Remote Devices**: No more relying on on-prem connections. WUfB and Intune manage updates wherever there’s internet, which is perfect for today’s hybrid and remote work setups.
    
2.  **Less Maintenance**: No more WSUS servers to manage, patch, and troubleshoot. Windows Update for Business taps into Microsoft’s cloud, taking that workload off your plate.
    
3.  **Control Without Micromanagement**: Update rings let you stagger deployments, creating a more reliable update process without micromanaging every patch.
    

The Power of Update Rings: A Gradual, Controlled Rollout
--------------------------------------------------------

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/43df4b56-44e5-4d57-ae5c-e7e37afb5068/part6-1.png)

With WSUS, you’d manually approve each update, pushing it out in waves across the organization. In Intune, **update rings** give you similar control but with more automation and flexibility. You can structure updates into rings, assigning devices to groups for a gradual rollout that minimizes disruption and keeps issues isolated to smaller groups.

Here’s how to set up an effective update ring structure:

-   **Pilot Ring**: Start with a small group of test devices. These are your first line of defense, helping you catch issues before they reach the wider organization.
    
-   **Fast Ring**: After updates pass the pilot phase, move to a slightly larger group of early adopters. They’ll help you ensure updates are stable on a broader range of devices.
    
-   **Broad Ring**: Finally, the remaining devices get updates. With pilot and fast rings giving you a head start on testing, the broad ring should be relatively smooth sailing.
    

This approach not only reduces the risk of widespread issues but also allows you to tailor the update experience to different user needs and device configurations.

### Configuring Update Rings: Timing, Deferrals, and Active Hours

One major perk of update rings is that they allow you to manage update timing without constant approvals. You can set **deferrals** and **active hours** to make sure updates happen at the right time for your organization.

-   **Deferral Policies**: Set deferrals on quality updates (security patches) and feature updates (like major OS releases). For security patches, aim for a short deferral, like a few days or a week. For feature updates, you can set a longer deferral period, giving time for testing.
    
-   **Active Hours**: Set active hours to avoid updates during high-traffic work hours. This minimizes disruptions and ensures devices don’t reboot in the middle of something critical.
    

With these settings, you maintain a level of control that feels familiar to WSUS but with less manual work, making it easy to keep updates predictable and user-friendly.

### Bonus: Using Update Rings to Streamline Windows 11 Upgrades

One of the often-overlooked benefits of moving to Windows Update for Business is how much easier it makes **upgrading to Windows 11**. With update rings, you have a smooth, structured path for deploying Windows 11, similar to managing feature updates in Windows 10.

Here’s how to use update rings to simplify your Windows 11 rollout:

-   **Pilot Group Testing for Compatibility**: Start by upgrading a small pilot group to Windows 11. This lets you test for compatibility with existing applications, drivers, and configurations, catching any issues before a full rollout.
    
-   **Staggered Deployment Across Rings**: Once the pilot ring is stable, move to the fast ring, gradually expanding the Windows 11 rollout. This helps avoid overwhelming support teams and gives users time to adjust.
    
-   **Controlled Timing with Feature Deferrals**: Since WUfB separates feature updates from quality updates, you can control exactly when Windows 11 rolls out. Set a deferral for non-pilot devices until you’re ready to roll out Windows 11 across the organization.
    

With this approach, the transition to Windows 11 feels like just another feature update—smooth, gradual, and far less stressful than the traditional, all-at-once OS upgrade.

Boost Efficiency with Delivery Optimization for On-Site Updates
---------------------------------------------------------------

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/47b52aff-e318-4d8a-b8a9-cf4718ef6c18/part6-2.png)

For on-site environments, Delivery Optimization in Intune is a game-changer for bandwidth management. Delivery Optimization allows devices to download updates from local peers, reducing the load on your network and speeding up the process for everyone.

### How Delivery Optimization Works

Instead of each device downloading updates directly from Microsoft, Delivery Optimization enables devices to share content with one another. Devices on the same network can pull update files from nearby peers, meaning fewer downloads from the internet and faster updates overall.

### Setting Up Delivery Optimization in Intune

1.  **Configure Download Mode**: In Intune, set the **Delivery Optimization download mode** to “Group” or “LAN,” which will enable devices on the same network to share update files.
    
2.  **Monitor Network Performance**: Delivery Optimization minimizes internet bandwidth usage, but you can monitor its effect using Intune’s built-in reports. Check the metrics to see how much bandwidth has been saved and if any adjustments are needed.
    
3.  **Adjust Based on Office Size**: If you have multiple office locations, you can adjust Delivery Optimization settings per location. Larger offices with more devices will see more benefit, while smaller offices might do fine with the default settings.
    

**The Result**: Faster updates, less strain on your internet connection, and a smoother experience for users—all with minimal effort on your part.

Compliance Policies: Making Sure Devices Are Up-to-Date
-------------------------------------------------------

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/62198685-a8ab-47da-a45d-06b17ef76944/part6-3.png)

Intune’s **compliance policies** are your best friend for ensuring devices stay updated. Compliance policies allow you to set requirements for OS and patch levels, so you can easily see if devices are falling behind.

-   **Monitor Compliance with Reports**: Intune’s reporting gives you a clear view of which devices are compliant and which need attention. It’s an efficient way to catch any stragglers.
    
-   **Conditional Access for Security**: If a device isn’t meeting update compliance, Conditional Access can block it from sensitive resources until it’s up to date. It’s a great way to keep data secure while motivating users to stay compliant.
    

Using compliance policies lets you focus on overall update health without constantly checking each device individually.

Testing and Fine-Tuning the Update Process
------------------------------------------

Moving away from WSUS doesn’t mean abandoning testing. If anything, the transition to Windows Update for Business makes it even more important to monitor and fine-tune the update process.

-   **Use Intune’s Reporting Tools**: Monitor update status and compliance across your rings to catch issues early. If an update fails in the pilot ring, hold off on rolling it out further.
    
-   **Adjust Rings Based on Feedback**: Pay attention to feedback from users in the pilot and fast rings. If issues crop up, make adjustments before proceeding to the broad ring.
    

Wrapping Up: A Smarter Way to Manage Updates
--------------------------------------------

Transitioning from WSUS to Windows Update for Business is a big change, but update rings provide a flexible, scalable approach to managing updates across your organization. With a structured rollout, deferred updates, compliance policies, and Delivery Optimization, you can keep devices secure and users happy—all while keeping your infrastructure light.

And with the added bonus of a streamlined path to Windows 11, you’re setting up your organization for an easy transition to Microsoft’s latest OS on your terms. In our next post, we’ll dive into optimizing compliance policies in Intune to keep your environment secure and consistent. Until then, happy updating!
