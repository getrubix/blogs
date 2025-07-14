---
author: GuestUser
date: Fri, 31 May 2024 12:48:03 +0000
description: '"Most organizations like to configure OneDrive to automatically sign-in
  with the user’s credentials. However, this doesn’t always work when multi-factor
  authentication (MFA) is enabled for their accounts – this may be intentional/unintentional
  due to MFA targeted to All Cloud Apps (the common example that I loathe, but I’ll"'
slug: onedrive-are-you-there-its-me-mfa-8C50m
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/onedrive-are-you-there-its-me-mfa-8C50m_thumbnail.jpg
title: OneDrive are you there Its me MFA
---

Most organizations like to configure OneDrive to automatically sign-in with the user’s credentials. However, this doesn’t always work when multi-factor authentication (MFA) is enabled for their accounts – this may be intentional/unintentional due to MFA targeted to All Cloud Apps (the common example that I loathe, but I’ll come back to that another time…), or it may be targeted just for the sake of protecting OneDrive in general. Either way, it would be nice to get OneDrive to still do its thing in this scenario.

You could of course look at adjusting your Conditional Access policies… perhaps you can grant access based on the device being Compliant (and still require MFA if it is not compliant). You could also look implementing Windows Hello for Business to help with some MFA-related prompts, however in some cases that is not an option – with this mind, how else can we ensure OneDive is configured for the end-user?

Custom solution
---------------

From an administrator perspective, we can of course track devices remotely with the OneDrive Sync Reports. When it comes to the end-users however, how do we make sure they are setting up OneDrive when they first get their system?

We can force them with a prompt upon login, of course. Here’s a PowerShell script that will create a Run key for the default user profile; you’ll want to assign this to devices so that it deploys during Autopilot (feel free to wrap it as a win32 app if you want to strictly track it via ESP):

reg.exe load HKLM\\TempUser "C:\\Users\\Default\\NTUSER.DAT" | Out-Host
reg.exe add "HKLM\\TempUser\\Software\\Microsoft\\Windows\\CurrentVersion\\Run" /v "LaunchOneDriveWithUser" /d "C:\\Windows\\LaunchOneDriveWithCurrentUser.cmd"
reg.exe unload HKLM\\TempUser | Out-Host

$scriptText = "@echo off > nul
for /f \`"delims=\`" %%n in ('whoami/upn') do set upn=%%n

REM determine if Business1 instance is configured on this machine
reg.exe query HKCU\\Software\\Microsoft\\OneDrive\\Accounts\\Business1 /v UserEmail > nul 2> nul
if errorlevel 1 goto :NotProvisioned
REM since OneDrive is provisioned already, dont run this script anymore
reg.exe delete HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Run /v LaunchOneDriveWithUser /f
goto :EOF
:NotProvisioned
REM there was no Business1 UserEmail registry entry so launch OneDrive first run wizard with the logged in user ID
start odopen://sync?useremail=%upn%
"

New-Item -ItemType File -Path "C:\\Windows" -Name "LaunchOneDriveWithCurrentUser.cmd" -Force
Add-Content -Path "C:\\Windows\\LaunchOneDriveWithCurrentUser.cmd" $scriptText | Set-Content "C:\\Windows\\LaunchOneDriveWithCurrentUser.cmd" -Force
Start-Sleep -Seconds 2

If you create an intunewin with this oneDrive\_SignIn.ps1 script, your install command and detection rule should be the following:  
  
**INSTALL COMMAND:** powershell.exe -executionpolicy bypass .\\oneDrive\_SignIn.ps1

**DETECTION RULE:** File - C:\\Windows\\LaunchOneDriveWithCurrentUser.cmd

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/af6bdf3f-260c-48ab-95e2-c1b8c5875ebb/Picture1.png)

Once the user is configured in OneDrive, the next time the .cmd file runs at log on, it will determine that the Run registry value is no longer needed (and it will run: reg.exe delete HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Run /v LaunchOneDriveWithUser /f)

One interesting thing to note – with this scenario, the user will technically see the option to enable or disable Known Folder Move (KFM). This may seem bad, but the good news is your policy enforcement of KFM will still take effect regardless of what the user selects. You can verify this by enabling the end-user notification when KFM is complete.

![Picture2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1717159317000-A072JJB183RQTCXLLBAV/Picture2.png)

![Picture2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1717159317000-A072JJB183RQTCXLLBAV/Picture2.png)

![Picture3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1717159316979-JXN44OBN3FBKMSV3U5DV/Picture3.png)

![Picture3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1717159316979-JXN44OBN3FBKMSV3U5DV/Picture3.png)

#block-yui\_3\_17\_2\_1\_1717159105607\_24693 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -20px; } #block-yui\_3\_17\_2\_1\_1717159105607\_24693 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 20px; margin-bottom: 20px; }

And that’s it. The review of Conditional Access and/or Windows Hello for Business configurations would be the first path to explore, but I like to test these workarounds when changes can’t be easily made in certain environments. Enjoy!
