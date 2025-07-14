---
author: steve@getrubix.com
date: Wed, 30 Oct 2024 19:00:25 +0000
description: '"If you’ve made it this far, congratulations! You have completed the
  first phase, known as the Discover phase. But wait—when did we mention phases?We
  didn’t! That’s the focus of today’s post.Don’t forget to follow the video companion
  for this series on the GetRubix YouTube members channel.A Repeatable ModelTechnically,"'
slug: getting-started-with-intune-part-3-the-phases
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/getting-started-with-intune-part-3-the-phases_thumbnail.jpg
title: Getting Started with Intune Part 3 The Phases
---

If you’ve made it this far, congratulations! You have completed the first phase, known as the **Discover** phase. But wait—when did we mention phases?

We didn’t! That’s the focus of today’s post.

Don’t forget to follow the video companion for this series on the [GetRubix YouTube members channel](https://www.youtube.com/playlist?list=UUMOF6q8UjlE5AFO52ht-G_L6A).

A Repeatable Model
------------------

Technically, every implementation has phased components, and Intune is no exception. When my team and I approach an Intune project—whether it’s a full cloud-native Windows and Autopilot deployment or a configuration of app protection policies—we always follow the same method:

> **Discover, Design, Build, Adopt**

Let’s go through each phase and how it specifically relates to an Intune implementation.

### Discover

The **Discover** phase focuses on understanding the current environment. This was covered in [Part 1](https://www.getrubix.com/blog/getting-started-with-intune-part-1-where-are-you-today), so I won’t go too deep here. However, during this phase, ensure the following:

-   Understand device types, quantities, use cases, and user personas.
    
-   Fully comprehend the management tools currently in use, including their configurations and day-to-day operations.
    
-   Identify the applications deployed to devices and the patching processes.
    
-   Determine if BYOD is allowed for mobile devices.
    
-   Be aware of any other environmental factors that might impact the Intune project.
    

### Design

We touched on the **Design** phase in [Part 2](https://www.getrubix.com/blog/getting-started-with-intune-part-2-dont-build-for-the-fringe), but there’s more to explore here.

With the insights from Discover, we can plan for Intune’s implementation—not just for onboarding devices but also by addressing:

-   Policies to configure
    
-   Applications to package and deploy
    
-   Windows Update for Business rings to create
    
-   Reviewing current GPOs and deciding between migration and starting fresh
    
-   Onboarding and enrollment flows, including Autopilot, Apple Business Manager, and Android Enterprise
    
-   Identifying users/devices for the initial pilot
    

For Design to be effective, create a comprehensive design architecture document. This will vary by project, but here’s an example from an Autopilot design I’ve used:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e111f216-28e8-435e-b4a0-d6e96ab6500c/NY.png)

A clear strategy for Autopilot devices with group tag based assignment

The document illustrates applications and policies deployed to device groups and details the grouping of devices for Autopilot using group tags. When working with mobile devices, for instance, I’ve designed flows to migrate users from their existing MDM to Intune, with key steps clearly documented.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/c0a79639-2d61-492b-abb2-4063da07d563/Blank+diagram+%281%29.png)

In cases where you’re doing something unique, like migrating PCs between Intune tenants, you might end up with a design that looks entirely different.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/48854b23-2ebb-4493-81c9-79a6093f0917/Device+Migration+V6.2+-+Overview.png)

Pure madness!

Whichever direction your design takes, having it in writing allows all stakeholders to review and “resolve issues on paper” before any configurations are made.

### Build

This phase should be straightforward. After completing Discover and finalizing the design document, it’s time to **Build**.

In this phase, all policy configurations, application packaging, PowerShell scripting, and general implementation happen. Though the official pilot follows later, I strongly recommend enrolling a test device or virtual machine for real-time troubleshooting during the build process.

Build based on the design document, keeping detailed notes along the way. Often, it’s useful to produce an “as-built” document that captures all configurations at completion, serving as both a reference and a point-in-time snapshot should any rollback be needed.

I recommend this tool for documenting an Intune environment:

[Automatic Intune Documentation Tool](https://www.wpninjas.ch/2021/05/automatic-intune-documentation-evolves-to-automatic-microsoft365-documentation/)

### Adopt

Finally, we’re at the end—or are we just beginning?

Welcome to the **Adopt** phase.

I recommend a multi-step pilot program for each use case being tested. After each pilot wave, address any configuration issues. By the final pilot, you should have the maximum number of devices enrolled with minimal issues, ready for fleet-wide deployment.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/b8f37fa8-57ad-4b3c-9d9d-14a80953f288/Blank+diagram+%282%29.png)

As the device count goes up, the number of issues go down.

Just One Cycle?
---------------

The four phases are not always linear. This is a framework to guide you, even as you add new components long after Intune is in place. Remember in [Part 2](https://www.getrubix.com/blog/getting-started-with-intune-part-2-dont-build-for-the-fringe) when we discussed avoiding “fringe devices”? Without going through Discover, you wouldn’t even identify those devices, and Design would be the ideal time to strategize on handling them.

For instance, after completing Build and Adopt for one or two use cases, a full design architecture allows you to seamlessly move back into Build and Adopt for additional configurations.

No matter how complex the transition to Intune may seem, following these phases can simplify the process.
