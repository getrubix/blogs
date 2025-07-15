---
author: steve@getrubix.com
date: Tue, 14 Apr 2020 14:05:43 +0000
description: 'For three years now, I have led a team of engineers in the practice
  of deploying modern management for Windows 10. Last month, when the pandemic induced
  lock-down started, the world of Windows users lost their line-of-site to their corporate
  domains.'
slug: covid-co-management-crisis
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/covid-co-management-crisis_thumbnail.jpg
title: Covid Co-Management Crisis
---

For three years now, I have led a team of engineers in the practice of deploying modern management for Windows 10. Last month, when the pandemic induced lock-down started, the world of Windows users lost their line-of-site to their corporate domains. The modern management my team had evangelized for years had suddenly pivoted from being an eventual destination to an urgent need. This is not the push to modern management we wanted to see.

Most businesses cannot shift to Autopilot overnight (my record in setting up a production environment for a customer sits at 4 days). It takes time to build the profiles in Intune, repackage applications, moving group policy to admin templates; typically anywhere from 4-6 weeks.

The good news here is that if you implement SCCM, you can quickly enable co-management to get control of your spontaneously, remote workforce. While I've not been a fan of it in the past, it's hard to deny that this is the best path forward for most right now. There are two flavors of co-management depending on the scenario: current fleet or new devices. And within current fleet, there are two variants. Today we'll look at the first one; co-management for domain join devices with SCCM and Intune.

## Current fleet and Intune
---

>I understand that most of your users, if not all, are currently not connected to the domain. To set up co-management for them, they will need to VPN in at least until we've deployed the appropriate changes to their SCCM client.

Without a doubt, this is the easier scenario. Essentially, we’ll be enrolling SCCM managed devices to Intune and pushing as much or as little of the workload as we want there. Here are the most relevant workloads to help immediately:

![2020-04-14 09_39_33-CONFIGMAN (configman3000.zerotouch.local) - Remote Desktop Connection Manager v2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/593971df-a139-409c-a6f7-2c27fedcf154/configman.png)

-   **Office 365 deployment**: deploy and manage the Office suite including individual apps, update channels, and configuration policy (admin templates)
    
-   **Windows Update for business**: enforce policy based on the Windows update ring method so users do not need to rely on WSUS to receive updates while they're remote. Updates are deployed directly from the Microsoft CDN
    
-   **Endpoint Protection**: This includes settings for encryption, local security and Windows Defender.
    
-   **Client Apps**: allow Intune to deploy apps to devices as either required or available through the company portal.
    
## Architecture
--- 

Here is a high-level overview of the architecture:

![2020-04-13 23_43_46-Co Management.pptx - PowerPoint.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1586871708305-TZ26HGLJEUQQF8U05IVB/2020-04-13+23_43_46-Co+Management.pptx+-+PowerPoint.png)

You can see that even though the device is domain bound and still managed by the client, you now have the ability to push additional applications and configurations from Azure. These require no line-of-sight to the domain, thus providing an immediate fix for managing a remote workforce.

Enabling co-management in SCCM is fairly straight forward and is best detailed in the infamous "**4 clicks to Co-Management**" by Brad Anderson found [here](https://techcommunity.microsoft.com/t5/microsoft-endpoint-manager-blog/co-management-is-instant-and-easy-with-just4clicks/ba-p/250539).

Here is a breakdown of the steps:

1. Hybrid Azure AD join must be enabled. This allows domain joined devices to be visible in Azure and eligible for Intune enrollment
    

![2020-04-14 09_07_03-z0tdc02.zerotouch.local - Remote Desktop Connection.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1586871814535-MI7M0IT0KBL6UA8W9J4G/2020-04-14+09_07_03-z0tdc02.zerotouch.local+-+Remote+Desktop+Connection.png)

2. Assign the EMS (Enterprise Mobility + Security) licensing to all applicable users and enable Automatic Enrollment in Intune for Windows devices.
    

![2020-04-14 09_44_51-Chuck Berry _ Licenses - Microsoft Azure.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1586871900872-MRRNQWEKHZE7PHZTMCB7/2020-04-14+09_44_51-Chuck+Berry+_+Licenses+-+Microsoft+Azure.png)

3. In the SCCM console, navigate to **Administration > Cloud Services > Co-management** and click “**Configure co-management**”.
    

![2020-04-14 09_36_10-CONFIGMAN (configman3000.zerotouch.local) - Remote Desktop Connection Manager v2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/593971df-a139-409c-a6f7-2c27fedcf154/comag_console.png)

4. Sign in to Azure and select your pilot group for enrollment.
    

![2020-04-13 22_30_37-configman3000.zerotouch.local - Remote Desktop Connection.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1586872033328-4PL9M7WKA0SQRAFCZJCR/2020-04-13+22_30_37-configman3000.zerotouch.local+-+Remote+Desktop+Connection.png)

## Workloads
---

From there, you can enable the correlating features in Intune that match up to the workloads you'll be shifting. The cool part is that even without shifting workloads, you gain access to the following features:

-   **Remediation actions:** Remote wipe, lock, restart and sync with the device.
    
-   **Client health**: Get visibility to current state and health of the PC, including encryption and compliance status.
    
-   **Conditional access:** Immediately put conditional access into play to further secure access to corporate resources. For instance, you can require Windows devices to be marked as compliant in order to access Office 365, OneDrive, and SharePoint.
    

It's safe to say that with some dedicated time and attention, you should be able to get this scenario up and running within a 2 to 3 day timespan. While it may not solve for every issue caused by the sudden influx of remote users, it is absolutely better than nothing.

Next up, we'll look at enabling the cloud-management gateway, which will provide current and new PCs with SCCM client access via the internet.
