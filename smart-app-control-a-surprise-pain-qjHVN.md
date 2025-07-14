---
author: GuestUser
date: Tue, 20 Aug 2024 15:13:45 +0000
description: '"Here’s a little feature I forgot about until it, well, caused an issue.
  A current customer of mine uses a third-party AV/EDR solution. With this in mind,
  they currently have policy to disable the Defender service (they are planning on
  enabling the service soon so that passive mode"'
slug: smart-app-control-a-surprise-pain-qjHVN
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/smart-app-control-a-surprise-pain-qjHVN_thumbnail.jpg
title: Smart App Control A Surprise Pain
---

Here’s a little feature I forgot about until it, well, caused an issue.

A current customer of mine uses a third-party AV/EDR solution. With this in mind, they currently have policy to disable the Defender service (they are planning on enabling the service soon so that passive mode can do it’s thing). In recently testing Windows 11 23H2 systems for Windows Autopilot, they encountered a couple of issues with installing their VPN software, as well as other EXE packages.

In the midst of troubleshooting and nearly reaching out to the vendor, they observed that MpCmdRun.exe and MsMpEng.exe were constantly starting and stopping (it would sometimes show 20+ instances running at once). Not only this, but the team also mentioned other recent test systems had general slowness and even overheating on one or two models.

 What’s interesting is when they ran Get-MpComputerStatus in PowerShell, it shows that Defender is not running. This at least lines up with the disablement policy… but maybe it was a different policy in Intune. We did have one Smartscreen policy to block/prevent bypass in Microsoft Edge, but turning it off did not seem to help. Perhaps Tamper Protection? Turns out this was off already… so what gives?

When reviewing the Microsoft doc on Defender compatibility with other security products, there is an interesting note for Windows 11:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/f477a422-5dc4-40a8-ab1e-936aba9f1eaa/blogtable1.png)

Ok. If we look further down in the document, it summarizes the following for Smart App Control State (in this scenario, we meet the criteria of the last line):

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/4332eeee-0cdb-43aa-be49-e2d81e3f478a/blogTable2.png)

So if using a third-party AV solution without Defender for Endpoint, Smart App Control will actually flip to Evaluation, or even On (yet Defender AV will disable automatically). Microsoft also has a separate document regarding WDAC and Smart App Control that states the following:

> “Smart App Control is only available on clean installation of Windows 11 version 22H2 or later, and starts in evaluation mode. Smart App Control is automatically turned off for enterprise managed devices unless the user has turned it on first. ”

What exactly is enterprise managed – domain or Entra joined? Utilizing either Defender AV and/or Defender for Endpoint? Based on the previous table I would guess the latter.

The document does provide instructions on disabling the setting via Registry, which is exactly what was tested next:

To turn off Smart App Control across your organization's endpoints, you can set the **VerifiedAndReputablePolicyState** (DWORD) registry value under HKLM\\SYSTEM\\CurrentControlSet\\Control\\CI\\Policy as shown in the following table. After you change the registry value, you must either restart the device or use [CiTool.exe -r](https://learn.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/operations/citool-commands#refresh-the-wdac-policies-on-the-system) for the change to take effect.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/d22352b9-5d04-4283-b727-0121a1420ad4/blogTable3.png)

Lo and behold, setting this registry value to 0 seemed to resolve the installation and performance issues across the board. We also had recalled some weirdness with running some privilege management software, as well as some PowerShell scripts a while back – perhaps this was part of that behavior?

One thing to be aware of - the document also has one of those blue-colored, important messages:

> “ Important:  
> Once you turn Smart App Control off, it can’t be turned on without resetting or reinstalling Windows.  
> ”

For an Autopilot Entra ID Joined system that is absolutely going to be using the third-party AV/EDR, I think this is acceptable for now. If moving to Defender AV/Defender for Endpoint later on, and you want to implement app controls without resetting the system, you can push WDAC policy later on from Intune.

Looks like a Remediation is in order. Hopefully that was helpful in case you are running into any installation/deployment issues with Windows 11.
