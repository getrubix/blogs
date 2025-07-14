---
author: steve@getrubix.com
date: Thu, 07 Nov 2024 14:46:29 +0000
description: '"So far, we’ve explored evaluating your environment, choosing initial
  use cases, planning and building your deployment, and the daunting process of moving
  Group Policy objects into Intune.But ultimately, you’re going to realize there is
  a challenge ahead; how are you going to deal with managing your new, shiny"'
slug: getting-started-with-intune-part-5-two-paths
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/getting-started-with-intune-part-5-two-paths_thumbnail.jpg
title: Getting Started with Intune Part 5 Two Paths
---

So far, we’ve explored evaluating your environment, choosing initial use cases, planning and building your deployment, and the daunting process of moving Group Policy objects into Intune.

But ultimately, you’re going to realize there is a challenge ahead; how are you going to deal with managing your new, shiny Autopilot devices next to your current, hybrid joined fleet? Do you handle both environments separately? Does help-desk need to figure out where each device is managed before assisting an end user, thus leading to them slowly losing their sanity?

No. Luckily, I’ve thought this through with something called the **“Two Paths”** approach—one for building your future, and one for bringing your past along for the ride.

Remember, there is a full companion video series to these blogs in the GetRubix YouTube [member channel](https://www.youtube.com/playlist?list=UUMOF6q8UjlE5AFO52ht-G_L6A).

### The Two Paths Strategy

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/49420d6e-e92d-4480-a445-b01c736dbac9/robot.png)

When integrating Intune and Autopilot into a landscape where hybrid joined or co-managed devices are already in play, many organizations struggle to find the balance between onboarding new devices and retrofitting existing ones. The answer? Start by focusing on the new devices and work backwards from there.

### Why Start with Autopilot?

You might be thinking, “Why bother with Autopilot first? Why not just tackle the existing fleet head-on?” Here’s the deal:

-   **Prove it out**: Configuring Intune for an Autopilot scenario serves as your real-world test environment. If you can build an effective, Entra Joined PC from scratch using only Intune, you know your policies and apps are stable.
    
-   **Iterative improvements**: Working with new devices gives you the freedom to make mistakes, learn, and iterate without impacting your existing user base. By the time you’re ready to onboard your current devices, you’ve already smoothed out the rough edges.
    
-   **Cloud native**: Embracing Autopilot helps your team adopt a modern approach. This isn’t just a technical shift—it’s cultural. Teams start to think in terms of cloud-first policies and configurations, which ultimately makes them more flexible when tackling the existing environment.
    

With that understanding, I present Path 1.

**Path 1: Build for the new**

-   **Focus on Autopilot**: Configure your apps, policies, and settings in Intune with the aim of supporting a net-new, Autopilot-provisioned Entra Joined PC. Why? Because if your environment works flawlessly for an Autopilot deployment, it’s robust enough to support your existing hybrid devices.
    
-   **Simplify policy**: By developing your Intune policies with a new Autopilot deployment in mind, you’re forced to take a hard look at what’s essential. Start with a minimal, well-documented set of configurations—think security baselines, core apps, and key compliance policies.
    
-   **Stress test**: Have your security teams run tests and audits on the new build. If those tests satisfy their concerns, then you’re good to go when the time comes to onboard the current fleet. If not, you know what you still have to build.
    

### Onboarding Hybrid Joined Devices to Intune: What to Expect

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7037e50f-4458-49c4-ba35-40fe7ca7c5ca/the-avengers-marvel.gif)

Once you’ve nailed down your new Autopilot deployment, the real fun begins. It’s time to onboard those hybrid beasts. Here’s what to expect:

-   **Automate it**: Depending on your setup, you should be able to automate this. The Group Policy to enroll hybrid devices to Intune is seamless and, in most cases, users don’t know what hit them. If you do need any manual user intervention, make sure to communicate clear and often.
    
-   **Policy Reconciliation**: This is where you thank your past self for starting with Autopilot. If you’ve done it right, the policies in place are comprehensive and stable, and any adjustments needed for hybrid joined devices will be minor.
    
-   **Compliance Shifts**: Devices that were once managed by GPOs now need to follow your brand-new Intune policies. Watch compliance reports carefully as you onboard devices to ensure they’re conforming as expected.
    

Excited yet? Good. Time to start the second path.

**Path 2: Bring the rest**

-   **Enroll the Hybrid devices**: Once you’ve validated your new build and proven that Intune can support a net-new device, it’s time to turn to your existing fleet. Start onboarding hybrid joined devices into Intune, confident that your policies and configurations are solid. Make sure to block inheritance from the on-premises GPOs so you minimize conflicts against Intune policy.
    
-   **Co-Management Strategy**: Use co-management to phase in Intune capabilities over time. Gradually shift workloads from ConfigMgr (or whatever on-prem management tool you’ve been clinging to) to Intune. Start with less disruptive workloads, like compliance policies and endpoint protection, and then move up to more complex ones. Windows Update is always an easy win, especially to help move to Windows 11.
    
-   **Unified support**: The end goal is that your help desk only has to deal with Intune for managing both new and existing devices. No more split-brain management where they need to toggle between different tools and interfaces. Just Intune. Clean, consistent, and centralized.
    

### Getting Help Desk on Board

Your help desk is going to be your biggest ally or your biggest obstacle, depending on how you handle this transition. Here’s how to make sure it’s the former:

-   **Start training**: Don’t wait until everything is set up to loop in your support teams. Get them acquainted with the Intune console and the common tasks they’ll be performing as soon as you have your first working Autopilot deployment.
    
-   **Create runbooks**: Document, document, document. From how to handle device enrollment hiccups to troubleshooting compliance issues, give your help desk detailed guides so they feel prepared and not left scrambling.
    
-   **Feedback loop**: The people on the front lines will have invaluable insights into what works and what doesn’t. Make sure there’s a mechanism for them to share feedback and that changes can be made quickly based on their experiences.
    

### Single pane of gl…

I know, we’re all sick of that phrase.

The goal of this “Two Paths” approach is to create a management environment that supports both new, Autopilot-deployed devices and the existing fleet—all from a single…eh…unified platform. It simplifies life for your help desk, streamlines policy management, and gives you a solid foundation for whatever comes next in the ever-changing world of IT.

How to handle both device types might be the biggest question you and your organization has about adopting Intune. Now, hopefully, you have an answer.
