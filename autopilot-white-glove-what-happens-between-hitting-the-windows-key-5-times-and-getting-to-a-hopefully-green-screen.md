---
author: steve@getrubix.com
date: Wed, 26 Aug 2020 02:19:11 +0000
description: 'There is a pattern I am starting to see when a company implements Autopilot
  and Intune to deploy Windows 10 PCs.&nbsp; At first, things seem to be going great-
  enrollments are successful, apps are deploying, policy is applied… just as intended.&nbsp;'
slug: autopilot-white-glove-what-happens-between-hitting-the-windows-key-5-times-and-getting-to-a-hopefully-green-screen
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-white-glove-what-happens-between-hitting-the-windows-key-5-times-and-getting-to-a-hopefully-green-screen_thumbnail.jpg
title: Autopilot White-Glove What happens between hitting the Windows key 5 times
  and getting to a hopefully green screen
---

There's a pattern I'm starting to see when a company implements Autopilot and Intune to deploy Windows 10 PCs.  At first, things seem to be going great- enrollments are successful, apps are deploying, policy is applied… just as intended.  So then, that company wants their hardware vendor to implement Autopilot White Glove for them.  Suddenly, what was meant to be a streamlined deployment process turns into an error filled, red-screened, nightmare with no explanation as to why things failed.

![red.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1598366352602-X3BUB6QK8WD1CPIU02QN/red.png)

## What is white glove?
---

For the uninitiated, Autopilot White Glove is the process of a PC being powered on, connected to the internet, and then run through an Azure AD Join / Intune enrollment by itself before the end user takes possession.  It's a great option as it takes care of the "heavy lifting" downloads, so the end user is waiting for 5 minutes after signing in as opposed to 30-60 minutes.  Here is a quick break down of the flow (officially known as the [_technician workflow_](https://docs.microsoft.com/en-us/mem/autopilot/white-glove#technician-flow)):

![flow-chart.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1598366397858-GALU5TPE2AT7U6RMYZUQ/flow-chart.png)

1. Autopilot registered PC is powered on and connected to network
    
2. Technician hits Windows key 5 times
    
3. Device communicates with Autopilot and determines which organization it belongs to along with which Autopilot profile it will receive.
    
4. PC receives all assigned applications and policy before finishing enrollment
    
5. Technician clicks the "Reseal" option on a successful, green screen, thus placing the PC back into the out-of-box state.
    

So if standard Autopilot works, why would it fail using White Glove?  Let's take a look at what happens after hitting that Windows key 5 times and what it takes for a successful enrollment.

![white-glove-result.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1598366884504-595ZT2S5I2BOVNCG9GKI/white-glove-result.png)

We’re going to use the Enrollment Status Page (ESP) and its three steps as a visual guide for White Glove. During provisioning, the **Device preparation** and **Device setup** sections need to finish in order to reach the successful green screen.  The **Account setup** portion is skipped via policy as this is configured after the user signs in later.**\***

**_\*_**_Yes, there is a workflow for performing White Glove on user-assigned devices, but this is not as common and not recommended in most situations.  For those still reading this tiny disclaimer, I will dive into this in a future post._

![esp-1903.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1598367001405-WVQQTQ6TIOI9TODM5CCM/esp-1903.png)

## Device preparation
---

Device preparation is a bit different in White Glove vs User-driven Autopilot.  Because the flow is started without user-affinity, it is incredibly similar to a self-deploying model. In order for a device by itself to register in Azure AD, there are some requirements.

-   Windows Auto-enrollment needs to be scoped to "All" as opposed to specific user groups.  Again, there is no user at this point so we have to let all devices in.  More on that [here](https://www.getrubix.com/blog/automatic-enrollment-dont-be-afraid).
    
-   TPM 2.0 must be present so that the PC can use a device token based off the health attestation to register with Azure AD
    
-   Use a fairly "open" network to conduct the provisioning. If your vendor's network is behind a firewall and cannot reach the appropriate enrollment URLs, things will fall down pretty quickly.  This includes all network requirements listed [here](https://docs.microsoft.com/en-us/mem/autopilot/networking-requirements), and if you're utilizing co-management, this needs to include your Cloud Management Gateway URL.
    

## Device setup
---

This is where the bulk of the work is done.  Towards the end of the previous section, Intune pushes the Intune Management Extension (IME) to the device, which is responsible for deploying applications and scripts to the PC.  When **Device setup** begins, the IME receives the ESP instructions as to which applications it needs to wait for before this section is considered complete.  I went into extremely gory details about this a [few posts back](https://www.getrubix.com/blog/please-wait).  But the two most important things are:

-   Proper assignment.  I cannot stress enough the need to plan for a successful White Glove enrollment by making sure policy and applications are assigned to the correct device groups.  Typically, these are the same dynamic groups that the Autopilot deployment profile gets assigned to.  Any applications not assigned to this group or assigned to a user group will not be provisioned during White Glove.
    
-   Make sure your applications are air-tight.  That means accurate install commands, perfect detection rules, well-written install scripts- everything to make sure there are no failures during ESP that will ultimately trip-up the White Glove enrollment.
    

## Account setup- (skip)
---

As a rule, I always recommending disabling the Account setup / User status tracking in ESP.  It has always been somewhat 'wonky' to me, and setup seems to always be smoother without it.  During a White-Glove provision, it's required, as there is no user object to track.  You can disable this with a custom configuration CSP:

-  `OMA-URI:` ./Vendor/MSFT/DMClient/Provider/MS DM Server/FirstSyncStatus/SkipUserStatusPage
    
-  `Data type:` Boolean
    
-  `*Value:` True
    

Assign this to the Autopilot dynamic device group mentioned previously.

Reaching a successful White Glove enrollment can seem like trial and error. And while there are exceptions and fringe cases out there, most obstacles can be overcome by following some of these guidelines. Do keep in mind, that everything covered here pertains to Azure AD join only. In the next write-up, we’ll explore White Glove with Hybrid Azure AD join, which yes- presents a whole additional set of challenges.
