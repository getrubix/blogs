---
author: GuestUser
date: Tue, 05 Mar 2024 16:49:07 +0000
description: '"I recently came across a question online regarding notification and
  Intune… specifically if the user decides to go to Settings -&gt; System -&gt; Notifications,
  and disables notifications from the Intune Management Extension, what is the impact?
  (NOTE: This setting is the same in both Win10 and Win11):"'
slug: intune-management-extension-notifications-keep-them-on-fzGXv
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/intune-management-extension-notifications-keep-them-on-fzGXv_thumbnail.jpg
title: Intune Management Extension Notifications Keep Them On
---

I recently came across a question online regarding notification and Intune… specifically if the user decides to go to **Settings -> System -> Notifications**, and disables notifications from the Intune Management Extension, what is the impact? (NOTE: This setting is the same in both Win10 and Win11):

![1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1709656602039-V3H0SEJZZ8J74R7TDBUN/1.png)

![1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1709656602039-V3H0SEJZZ8J74R7TDBUN/1.png)

![2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1709656602041-20JDD4VMREJW22A3BGC4/2.png)

![2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1709656602041-20JDD4VMREJW22A3BGC4/2.png)

#block-yui\_3\_17\_2\_1\_1709656527672\_14738 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -18px; } #block-yui\_3\_17\_2\_1\_1709656527672\_14738 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 18px; margin-bottom: 18px; }

Well, if you occasionally push applications to users, do you like them to be aware of these installs? Maybe not for ALL of the packages, but for some of them at least? Especially if there’s a potential for a brief slowdown, or an obvious change the user will see in the start menu or the desktop.

Luckily, there is a user registry entry we can track, which only  gets created when we disable notifications for the IME. Let’s take a look at HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Notifications\\Settings before and after we make the change:

![3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1709656673187-06XMTCDUGNB8123DOKAG/3.png)

![3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1709656673187-06XMTCDUGNB8123DOKAG/3.png)

![4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1709656673194-TUVKYA5VHHPAFYHVK0YC/4.png)

![4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1709656673194-TUVKYA5VHHPAFYHVK0YC/4.png)

#block-yui\_3\_17\_2\_1\_1709656527672\_25058 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -20px; } #block-yui\_3\_17\_2\_1\_1709656527672\_25058 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 20px; margin-bottom: 20px; }

This Notification setting did also make me curious… if I want to send an **Organizational message** from Intune – does this setting for IME affect that feature on my Win11 devices? If we look at Microsoft’s documentation, it might? Maybe?

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/32e54c31-6c3c-411e-8d3a-28038c3e1046/5.png)

That makes me more concerned about Do Not Disturb, which is technically enabled right after Autopilot enrollment completes (Microsoft calls that “quiet period”, but it gets disabled fairly quickly). There’s a granular switch for Notifications under the Intune Management Extension, which enables showing important notifications when Do Not Disturb is enabled, but we’ll come back to that later 

But, do Organizational messages even go through the IME? I assumed they were driven by a separate mechanism. Still, I briefly tried sending a message on a Win11-23H2 devie with the following policies enabled:

1.   **Enable delivery of organizational messages (User)**
    
2.  **Windows Spotlight – Enabled (I kept most sub-settings on, except for Third Party Suggestions and Spotlight On Lock Screen)**
    
3.  **Disable Cloud Optimized Content – Disabled**
    
4.  **Enable delivery of organizational messages (User) (Windows Insiders only)**
    

Ah shoot – that last one is still marked as _Windows Insiders only_ (even though the CSP documentation says it requires 10.0.22621.900). I’ll need to test this in a bit with an Insider Update Ring… for now, let’s just build the fix while I investigate.

Since this is a user registry path, we probably could just run and set things as the signed in user. However, I like to just run things as System; this way I don’t have to worry about access rights.

Here’s the solution:

### DETECTION:

```
$errorActionPreference = 'SilentlyContinue'

#Load users registry hive
New-PSDrive -PSProvider Registry -Name HKU -Root HKEY_USERS | Out-Null

#Get logged on user's SID
$userName = (Get-WmiObject -Class Win32_ComputerSystem | Select-Object UserName).UserName
$userSID = (New-Object System.Security.Principal.NTAccount($userName)).Translate([System.Security.Principal.SecurityIdentifier]).Value
$regPath = "HKU:\$userSID\Software\Microsoft\Windows\CurrentVersion\Notifications\Settings\Microsoft.Management.Clients.IntuneManagementExtension"

#Get the registry property
$value = Get-ItemProperty -Path $regPath | Select-Object -ExpandProperty "Enabled"

if($value -eq 0)
{
    Write-Host "Notifications disabled - running remediation..."
    Exit 1
}
else
{
    Write-Host "Notifications enabled"
    Exit 0
}
```

### REMEDIATION:

```
$errorActionPreference = 'SilentlyContinue'

#Load users registry hive
New-PSDrive -PSProvider Registry -Name HKU -Root HKEY_USERS | Out-Null

#Get logged on user's SID
$userName = (Get-WmiObject -Class Win32_ComputerSystem | Select-Object UserName).UserName
$userSID = (New-Object System.Security.Principal.NTAccount($userName)).Translate([System.Security.Principal.SecurityIdentifier]).Value
$regPath = "HKU:\$userSID\Software\Microsoft\Windows\CurrentVersion\Notifications\Settings\Microsoft.Management.Clients.IntuneManagementExtension"

#Get the registry property
Remove-ItemProperty -Path $regPath -Name "Enabled" -Force
```

Another thing… before I mentioned an option related to do not disturb, within the notification settings for the IME:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/56940a36-1b05-4395-9e9e-03cadd04c464/6.png)

It turns out if you turn on “Allow app to send important notifications when do not disturb is on”, an additional registry key is created in our HKCU path:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/566c584c-7ab2-4198-ad0a-19d06c7af367/7.png)

That setting is important as well, so let’s amend our remediation package to fix that setting:

Variables and If statement changes:

```
#Get the registry property
$enabled = Get-ItemProperty -Path $regPath | Select-Object -ExpandProperty "Enabled"
$urgent = Get-ItemProperty -Path $regPath | Select-Object -ExpandProperty "AllowUrgentNotifications"

if($enabled -eq 0 -or $urgent -ne 1)
{
    Write-Host "Notifications disabled - running remediation..."
    Exit 1
}
else
{
    Write-Host "Notifications enabled"
    Exit 0
}


And the remediation update:

#Set the registry properties
Remove-ItemProperty -Path $regPath -Name "Enabled" -Force
New-ItemProperty -Path $regPath -Name "AllowUrgentNotifications" -Value 1 -Force
Write-Host "Registry values updated."
```

This gave me one last idea…can we set that urgent notifications value in Niehaus’s Autopilot Branding package? Will this allow those IME toasts to show even during the initial the “quiet period” after Autopilot? 

Interestingly, there is also another registry path I observed in HKLM:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e20bf3f7-1778-4999-82ca-6da4238dc708/8.png)

Maybe we should set the HKLM and HKCU entries? Here’s a potential addition to the end of your AutopilotBranding.ps1:

```
# STEP 16: Misc user defaults
Log "Enforcing IME notifications"
reg.exe add "HKLM\ SOFTWARE\Microsoft\Windows\CurrentVersion\Notifications\QuietHours\AlwaysAllowedApps" /v "IntuneManagementExtension" /t REG_SZ /d "Microsoft.Management.Clients.IntuneManagementExtension" /f | Out-Host

reg.exe load HKLM\TempUser "C:\Users\Default\NTUSER.DAT" | Out-Host
reg.exe add "HKLM\TempUser\SOFTWARE\Microsoft\Windows\CurrentVersion\Notifications\Settings\Microsoft.Management.Clients.IntuneManagementExtension" /v "AllowurgentNotifications" /t REG_DWORD /d 1 /f | Out-Host
reg.exe unload HKLM\TempUser | Out-Host
```

Hopefully this helps alleviate concerns around end-user notifications. I still need to confirm the impact on Organizational Messages and the post-enrollment Quiet Hours, but at least we can ensure IME notifications will generally be enabled.
