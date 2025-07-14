---
author: steve@getrubix.com
date: Mon, 27 Jan 2025 21:55:03 +0000
description: '"UPDATE 1/28/25: Many thanks to Steven Hosking at Microsoft for pointing
  out a different CSP that I was missing. It turns out I tried everything but this
  Microsoft account CSP. I’d say this is the much simpler solution and to deploy this
  as a configuration profile instead!Original Post:"'
slug: kioskuser0-and-the-orange-dot-1rdM5
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/kioskuser0-and-the-orange-dot-1rdM5_thumbnail.jpg
title: Kioskuser0 and the Orange Dot
---

**UPDATE 1/28/25:** Many thanks to Steven Hosking at Microsoft for pointing out a different CSP that I was missing. It turns out I tried everything but [this Microsoft account CSP](https://learn.microsoft.com/en-au/windows/client-management/mdm/policy-csp-LocalPoliciesSecurityOptions#accounts_blockmicrosoftaccounts). I’d say this is the much simpler solution and to deploy this as a configuration profile instead!

**Original Post:** I know we wrote recently about kiosk systems and a [recent change with desktop icons](https://www.getrubix.com/blog/so-along-kiosk-desktop-icons). However, there’s another annoying issue that we’ve noticed as of late. If you see a dark orange dot on the account icon in the start menu, this is what you’ll see when you click on it:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/99d1d9b0-77d3-4368-9a7c-7f193dfce429/accountnotice1.png)

Now why on Earth would I want that for an auto-logon kiosk account? What’s interesting is when I click on it, the Windows Backup application opens with a window stating, “This feature is not supported by your organization.” Well that’s good at least… but what if I click the blue link on there?

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/5f3f6e93-fb74-4de4-ac9a-19d25651bbaa/accountnotice2.png)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/31ca452b-581b-4d7a-84e6-a2bfbf710a11/accountnotice3.png)

Oh boy, it opened a standard Edge window – this is not good for my particular build. I basically allow Edge and several other apps, though I have Edge set to auto-launch with some custom kiosk arguments (and I even have a custom .lnk in the start menu with the same settings to re-open it).

This is not good if this little blue link can bypass my desired kiosk state for Edge. I supposed I could counteract this for now by setting “Hide the First-run experience and splash screen” to **Enabled**, as well as setting “Configure InPrivate mode availability” to **InPrivate mode forced**.

### **The Policy**

While the Edge policies would help, wouldn’t it be better to just get rid of this notification? Are there any policies that can directly disable this annoyance?

At first, I wasn’t sure if this fell under blocking Microsoft accounts or blocking consumer MS account auth; I tried those and they unfortunately did not work for me. Then I found the following [CSP for account notifications](https://learn.microsoft.com/en-us/windows/client-management/mdm/policy-csp-notifications#disableaccountnotifications):

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/4bd6447f-d654-44f9-80e4-cd065455099b/accountnotice4.png)

Ah shoot… not only is this limited to 24H2, but it supports **user scope only** - that’s not really going to apply to a local/non-EID account. I guess another script is in order.

### **The Script**

We will need to set the following registry value to disable these notifications:

**Path:** HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced  
**Name:** Start\_AccountNotifications  
**DWORD Value:** 0

Now I could deploy this as a Remediation script through Intune… though I would have to do some tricks to get the current logged-on user as System. Also, the other consideration is the timing of the remediation – I could have it run hourly, but there’s still the possibility of it not hitting right away.

In my opinion, this would be best suited to deploy as a local **scheduled task**. I normally would even consider a Run command, but that is blocked in the kiosk mode.

SO – let’s deploy a platform script (or a script wrapped in a win32, if you want to track it during ESP) to plant the scheduled task:

* * *

\# Define task settings
$TaskName = "Disable Account Notifications"
$TaskAction = New-ScheduledTaskAction -Execute "C:\\Windows\\System32\\reg.exe" -Argument 'add "HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced" /v "Start\_AccountNotifications" /t REG\_DWORD /d 0 /f'
$TaskTrigger = New-ScheduledTaskTrigger -AtLogOn
$TaskPrincipal = New-ScheduledTaskPrincipal -GroupId "Users" -RunLevel Highest

# Define additional settings with Hidden checkbox set to true
$TaskSettings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries \`
-DontStopIfGoingOnBatteries \`
-StartWhenAvailable \`
-Compatibility 4 \`
-ExecutionTimeLimit (New-TimeSpan -Hours 1) \`
-Hidden

# Create the scheduled task
Register-ScheduledTask -TaskName $TaskName -Action $TaskAction -Trigger $TaskTrigger -Principal $TaskPrincipal -Settings $TaskSettings -Description "Disables account notifications for any user on login."

# Log file for .intunewin
$path = "C:\\ProgramData\\applogs"
$name = "kioskAccount\_NotificationsDisabled.txt"
If(!(Test-Path $path)) {
    mkdir $path
}
New-Item -ItemType File -Path $path -Name $name -Force
Add-Content "$($path)\\$($name)" "Notifications Disabled." | Set-Content "$($path)\\$($name)" -Force 

* * *

One last piece to this… since I need task scheduler and reg.exe to be able to do it’s thing, I will need to allow those two processes in my assigned access XML:

<AllowedApps>
      <App DesktopAppPath="C:\\Windows\\System32\\schtasks.exe />
      <App DesktopAppPath="C:\\Windows\\System32\\reg.exe />

There we go - that notification should now be gone. Hopefully one day the policy will be expanded to support device scope (or some kind of change is made behind the scenes for kiosk mode).
