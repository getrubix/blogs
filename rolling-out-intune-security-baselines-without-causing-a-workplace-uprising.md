---
author: Guest MVP
date: Mon, 17 Feb 2025 16:28:27 +0000
description: 'Rolling Out Intune Security Baselines Without Causing Office ChaosBig
  news Microsoft just dropped the latest version of Intune Security Baselines! These
  handy settings help lock down Windows devices, Microsoft Edge, and other Microsoft
  apps with best-practice security configurations. Think of them as a security cheat
  sheet, covering everything'
slug: rolling-out-intune-security-baselines-without-causing-a-workplace-uprising
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/rolling-out-intune-security-baselines-without-causing-a-workplace-uprising_thumbnail.jpg
title: Rolling Out Intune Security Baselines Without Causing a Workplace Uprising
---

Big news Microsoft just dropped the latest version of [Intune Security Baselines](https://learn.microsoft.com/en-us/mem/intune/protect/security-baseline-settings-mdm-all?pivots=mdm-24h2)! These handy settings help lock down Windows devices, Microsoft Edge, and other Microsoft apps with best-practice security configurations. Think of them as a security cheat sheet, covering everything from BitLocker encryption to Windows Defender policies and password rules basically, all the stuff that keeps IT folks up at night.

Now, I live and breathe CIS and NIST standards, so I get it security is crucial. But let’s be real: sometimes, even the best policies can cause unexpected chaos. Let me tell you about the time a security rollout _literally_ locked people out of a building.

## A Story about security turning physical

I was working with a massive customer, 87,000+ Windows devices and things were rolling out smoothly. Our pilot groups were around 7,000 devices at a time, no big deal. But then, there was this _one_ PC. Just one. Sitting under someone’s desk overseas, minding its own business… except it also happened to run the entire building’s badge system.

When the new security policies hit, that PC went down, and suddenly nobody could get into the building. _Oops._ Imagine a crowd of employees stuck outside, wondering why IT hates them. Luckily, because of how I roll out these policies, we quickly reversed the changes and saved the day (and probably a few friendships).

## A Smarter Way to Roll Out Security Baselines

This is exactly why I don’t believe in dumping an entire baseline onto devices all at once. Sure, it sounds efficient, but when something breaks, good luck figuring out _which_ setting caused the mess. Instead, here’s my method for a smooth, drama-free rollout:

1.  **Break It Down by Category**: Instead of pushing everything at once, I split settings into smaller groups and roll them out gradually. Less risk, easier troubleshooting. I’ve even packaged them up into individual .Json files for quick importing (link below!).
    
    [Security-Baselines/Windows Baseline 24H2 at master · dgulle/Security-Baselines](https://github.com/dgulle/Security-Baselines/tree/master/Windows%20Baseline%2024H2)
    >
    **UPDATE** This will not include the LAPS category setting for the Backup Directory. This setting does not show in the Settings Catalog._
    
## Demo of importing JSON into Intune

There are many methods to import .json files, but for this demo I’m just going to use the Intune Portal:

### Devices>Configuration> Create> Import Policy

  ![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/5d8fe554-38df-4436-a2c0-ca707801958e/blog1.png)

### Browse to .Json file> Assign a name for the policy>Click Save

  ![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/56026f20-b176-4eae-9766-8d04852e87fe/blog2.png)

### After creating the new policy, assign it as you normally would.

  
  >Use a UAT Form for Rollouts – The pace depends on how fast my customers want to move, but since some settings only change 1-2 things, the impact is usually low. I’ve provided a link to an example that I use below.

  [Security-Baselines/Windows Baseline 24H2/UAT Form.xlsx at master · dgulle/Security-Baselines](https://github.com/dgulle/Security-Baselines/blob/master/Windows%20Baseline%2024H2/UAT%20Form.xlsx)

## In closing
With this UAT form, I roll out settings to pilot groups. If users experience an issue, we identify that specific group and troubleshoot from there. This makes life _so much_ easier rather than debugging hundreds of settings at once, we’re only dealing with a handful at a time.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/139362a6-2319-4b0c-b33e-8690694cda70/blog3.png)

By taking this step-by-step approach, I keep security tight _without_ accidentally locking people out of their offices (again).
