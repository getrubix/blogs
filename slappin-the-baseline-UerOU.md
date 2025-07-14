---
author: steve@getrubix.com
date: Tue, 21 Jan 2025 15:54:28 +0000
description: '"Security Baselines… gotta love them. Well maybe not - I personally
  recommend breaking them out into Settings Catalog profiles; especially when you
  have a mix of other configuration profiles already in the mix (otherwise you may
  encounter some conflicts).However if you’re trying to see what policies Microsoft
  recommends,"'
slug: slappin-the-baseline-UerOU
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/slappin-the-baseline-UerOU_thumbnail.jpg
title: Slappin the MR Baseline
---

Security Baselines… gotta love them. Well maybe not - I personally recommend breaking them out into Settings Catalog profiles; especially when you have a mix of other configuration profiles already in the mix (otherwise you may encounter some conflicts).

However if you’re trying to see what policies Microsoft recommends, or if you are generally starting fresh with Windows device management in Intune, then the Baselines are a great starting point.  

### **HoloLens… is that you?**

Recently Microsoft announced that security baselines were in development for HoloLens 2 devices, and it looks like they made their way to Intune:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/b13fd1d3-038c-440c-ae95-02db43d0d6a0/hololens1.png)

Interestingly, Microsoft has not created a dedicated page yet with all of the [baseline settings](https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines). Also, I’m curious as to why Microsoft added this now since they recently announced the discontinuation of the HoloLens 2. The good news though is update support will continue through December 31, 2027, which means I can still test this thing for a while!

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/bf1cc8ca-0929-4be3-8280-082d2526208c/mehololensexport.jpg)

Now that’s a happy camper. Since there’s no official list yet of all the policies, I went ahead and looked through the Standard and Advanced lists and typed them all up below.

* * *

### **Standard Security Baseline for HoloLens 2**

Accounts
-	Allow Microsoft Account Connection: Block
Administrative Templates
    System > Power Management > Video and Display Settings
-	Turn off the display: Enabled – 30 seconds
Browser
-	Allow Cookies: Block only cookies from third party websites
-	Allow Password Manager: Block
-	Allow Smart Screen: Allow
Connectivity
-	Allow USB Connection: Not allowed
Device Lock
-	Device Password Enabled: Enabled
-	Alphanumeric Device Password Required: Password or Numeric PIN Required
-	Max Device Password Failed Attempts: Not configured
-	Max Inactivity Time Device Lock: 3 (minutes)
-	Device Password History: Not configured
-	Allow Simple Device Password: Not allowed
-	Device Password Expiration: Not configured
-	Min Device Password Length: 8
Experience
-	Allow Manual MDM Unenrollment: Block
Microsoft App Store
-	Allow All Trusted Apps: Explicit deny
-	Allow apps from the Microsoft app store to auto update: Allowed
-	Allow Developer Unlock: Explicit deny
Microsoft Edge
-	Block third party cookies: Enabled
-	Control which extensions cannot be installed: Enabled – Wildcard (\*)
-	Enable saving passwords to the password manager: Disabled
-	Configure Microsoft Defender SmartScreen: Enabled
Mixed Reality
-	AAD Group Membership Cache Validity in Days: 7
Settings
-	Allow VPN: Not allowed
-	Page Visibility List: hide:emailandaccounts;workplace;otherusers;bluetooth;usb;network-proxy;network-wifi;network-ethernet;network-airplanemode;powersleep;certificates;developers;windowsinsider;
System
-	Allow Storage Card: SD card use is not allowed and USB drives are disabled. This settings foes not prevent programmatic access to the storage card.
Tenant Lockdown
-	Require Network in OOBE (Device): true
Windows Hello for Business
-	Enable Pin Recovery: false
-	Restrict use of TPM 1.2: Disabled
-	Digits: Allows the use of digits in PIN
-	Expiration: 90 (days)
-	PIN History: 10
-	Lowercase Letters: Allowed
-	Maximum PIN Length: 6
-	Minimum PIN Length: 6
-	Special Characters: Allows the use of special characters in PIN
-	Uppercase Letters: Allowed
-	Require Security Device: true
-	Use Certificate for On Prem Auth: Disabled
-	Use Hello Certificates As Smart Card Certificates: Disabled
-	Use Windows Hello for Business (Device): true
Windows Update for Business
-	Allow Update Service: Allow
-	Manage Preview Builds: Disable Preview Builds

* * *

Not too shabby. I guess just be careful and review any current profiles prior to testing, especially around password, WHfB, and Windows Updates. If you’re using Update Rings currently, you should generally be ok if you’re not using an insider/dev channel. Here’s the bigger baseline:

* * *

### **Advanced Security Baseline for HoloLens 2**

Account Management
-	Deletion Policy: delete at both storage capacity threshold and inactivity threshold
-	Enable Profile Manager: True
-	Profile Inactivity Threshold: 30 (days)
-	Storage Capacity Start Deletion: 25 (percent)
-	Storage Capacity Stop Deletion: 50 (percent)
Accounts
-	Allow Microsoft Account Connection: Block
Administrative Templates
  System > Power Management > Video and Display Settings
-	Turn off the display: Enabled – 30 seconds
Browser
-	Allow Autofill: Block
-	Allow Cookies: Block only cookies from third party websites
-	Allow Do Not Track: Block
-	Allow Password Manager: Block
-	Allow Popups: Block
-	Allow Search Suggestions in Address Bar: Block
-	Allow Smart Screen: Allow
Connectivity
-	Allow Bluetooth: Disallow Bluetooth. The radio in the Bluetooth control panel will be grayed out and the user will not be able to turn Bluetooth on.
-	Allow USB Connection: Not allowed
Device Lock
-	Device Password Enabled: Enabled
-	Alphanumeric Device Password Required: Password or Numeric PIN Required
-	Max Device Password Failed Attempts: 10
-	Max Inactivity Time Device Lock: 3 (minutes)
-	Device Password History: 15
-	Allow Simple Device Password: Not allowed
-	Device Password Expiration: Not configured
-	Min Device Password Length: 12
Experience
-	Allow Manual MDM Unenrollment: Block
Microsoft App Store
-	Allow All Trusted Apps: Explicit deny
-	Allow apps from the Microsoft app store to auto update: Allowed
-	Allow Developer Unlock: Explicit deny
Microsoft Edge
-	Block third party cookies: Enabled
-	Configure Do Not Track: Disabled
-	Enable AutoFill for address: Disabled
-	Enable AutoFill for payment instruments: Disabled
-	Enable search suggestions: Disabled
-	Default pop-up window setting: Enabled - Do not allow any site to show popups
-	Control which extensions cannot be installed: Enabled – Wildcard (\*)
-	Configure a setting that asks users to enter their device password while using password autofill: Enabled – Autofill off
-	Enable saving passwords to the password manager: Disabled
-	Configure Microsoft Defender SmartScreen: Enabled
Mixed Reality
-	AAD Group Membership Cache Validity in Days: 7
Privacy
-	Let Apps Access Account Info: Force deny
-	Let Apps Access Account Info Force Allow These Apps: Microsoft.Dynamics365.Guides\_8wekyb3d8bbwe, Microsoft.MicrosoftRemoteAssist\_8wekyb3d8bbwe
-	Let Apps Access Background Spatial Perception: Force deny
-	Let Apps Access Background Spatial Perception Force Allow These Apps: Microsoft.Dynamics365.Guides\_8wekyb3d8bbwe, Microsoft.MicrosoftRemoteAssist\_8wekyb3d8bbwe
-	Let Apps Access Camera: Force deny
-	Let Apps Access Camera Force Allow These Apps: Microsoft.Dynamics365.Guides\_8wekyb3d8bbwe, Microsoft.MicrosoftRemoteAssist\_8wekyb3d8bbwe
-	Let Apps Access Microphone: Force deny
-	Let Apps Access Microphone Force Allow These Apps: Microsoft.Dynamics365.Guides\_8wekyb3d8bbwe, Microsoft.MicrosoftRemoteAssist\_8wekyb3d8bbwe
Search
-	Allow Search To Use Location: Block
Security
-	Allow Add Provisioning Package: Block
Settings
-	Allow VPN: Not allowed
-	Page Visibility List: hide:emailandaccounts;workplace;otherusers;bluetooth;usb;network-proxy;network-wifi;network-ethernet;network-airplanemode;powersleep;certificates;developers;windowsinsider;
System
-	Allow Storage Card: SD card use is not allowed and USB drives are disabled. This settings foes not prevent programmatic access to the storage card.
-	Allow Telemetry: Security
Tenant Lockdown
-	Require Network in OOBE (Device): true
Wi-Fi Settings
-	Allow Manual Wi Fi Configuration: Allow
Windows Hello for Business
-	Enable Pin Recovery: false
-	Restrict use of TPM 1.2: Disabled
-	Digits: Requires the use of at least one digits in PIN.
-	Expiration: 90 (days)
-	PIN History: 10
-	Lowercase Letters: Required
-	Maximum PIN Length: 6
-	Minimum PIN Length: 6
-	Special Characters: Requires the use of at least one special characters in PIN.
-	Uppercase Letters: Required
-	Require Security Device: true
-	Use Certificate for On Prem Auth: Disabled
-	Use Hello Certificates As Smart Card Certificates: Disabled
-	Use Windows Hello for Business (Device): true
Windows Update for Business
-	Allow Update Service: Allow
-	Manage Preview Builds: Disable Preview Builds

And that’s it. If you’re curious about what other Windows policies are supported for HoloLens 2 devices, take a peak at this additional Microsoft link: [Policies in Policy CSP supported by HoloLens 2 | Microsoft Learn](https://learn.microsoft.com/en-us/windows/client-management/mdm/policies-in-policy-csp-supported-by-hololens2). Don’t forget that Autopilot registration procedures are [available here](https://learn.microsoft.com/en-us/hololens/hololens2-autopilot#obtain-hardware-hash), as well as this helpful platform guide that combines the points above: [Use Windows Holographic devices with Microsoft Intune | Microsoft Learn](https://learn.microsoft.com/en-us/mem/intune/fundamentals/windows-holographic-for-business).
