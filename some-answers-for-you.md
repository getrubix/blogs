---
author: steve@getrubix.com
date: Sun, 21 Jul 2019 22:54:00 +0000
description: '"Most of my time during the work week is spent either configuring an
  Azure/Intune environment or talking about it. If it’s the first time introducing
  these modern concepts to a customer, they always have questions. And those questions
  are almost 99.99% the same questions, all the time. Someone"'
slug: some-answers-for-you
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/some-answers-for-you_thumbnail.jpg
title: Some answers for you
---

Most of my time during the work week is spent either configuring an Azure/Intune environment or talking about it. If it’s the first time introducing these modern concepts to a customer, they always have questions. And those questions are almost 99.99% the same questions, all the time. Someone I respect once told me if you have to do the same technical task more than 3 times, automate it. So I’m going to dedicate a few posts a week to answering those questions.

“What is Azure AD join and why am I not joining Windows 10 PCs to my local domain?”

Ah, the big one. This one gives everyone pause, though I cannot blame them. Not when Windows management has generally been done one way for thirty years. To answer this, let’s talk about why there is a need to domain join a PC in the first place. Here’s what domain join does:

-   PC is part of local network and has access to local resources such as print servers, file shares, and other things that live in your servers
    
-   When part of the local domain, PCs can receive GPO (Group Policy Objects), which is how most organizations restrict, control and manage their Windows machines
    
-   Being joined to the domain allows PCs to have line-of-sight to a domain controller and and SCCM (System Center Configuration Manager) distribution point. A company can use this to push applications, software packages and updates to PCs.
    

Wow. So with all that capability, why am I spending my days telling folks _not_ to join their PCs to the domain? Let’s examine the limitations of traditional domain join:

-   Control of PCs are limited to the local network. When users are remote (which is pretty common nowadays), there’s no control. Sure, GPO is still in place, but only those that have been previously configured. You can’t update policy on your network if the machine isn’t there.
    
-   Same goes for SCCM- software updates, application changes, asset tracking for inventory; all things that disappear the moment users are off network**\***
    
-   Deploying machines: not to completely beat a dead horse, but have I mentioned you need to be on the network to domain join? For new employees or users getting new PCs, domain joining means you must be on the network to setup a new PC. So if you’re deploying a machine to a remote user or swapping an executive’s machine because they’re on a two week business trip, then you are most certainly screwed!
    

**\***_Yes, it is possible to host an SCCM distribution point on the internet, but it’s generally not a good option in terms of security and cost._

Azure AD join occurs over the internet- no VPN, no tunneling somehow back to the local network; we’re talking straight up “sit yourself in a Starbucks and sign into public WiFi” internet. And shut up about security- employees use their standard AD (Active Directory) credentials to authenticate to Azure AD, which in turn authenticates to your local AD. There are several ways to do this, but the important thing to understand is that passwords are never stored in the cloud.

The bottom line is that when a PC is Azure AD joined, you have visibility and control over it no matter where it is. And once a PC is joined to Azure AD it can automatically be enrolled into Microsoft Intune which can replace both GPO and SCCM; yes, it can! But more on that later.
