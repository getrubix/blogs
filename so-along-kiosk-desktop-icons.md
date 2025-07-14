---
author: steve@getrubix.com
date: Tue, 07 Jan 2025 20:52:35 +0000
description: '"Recently, I encountered a challenging issue while setting up assigned
  access (kiosk) devices in Intune for a customer who manages multiple devices of
  this type. This customer relied heavily on desktop icons for their users - despite
  the availability of pinned list configurations, desktop shortcuts were a part"'
slug: so-along-kiosk-desktop-icons
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/so-along-kiosk-desktop-icons_thumbnail.jpg
title: So long Kiosk desktop icons
---

Recently, I encountered a challenging issue while setting up assigned access (kiosk) devices in Intune for a customer who manages multiple devices of this type. This customer relied heavily on desktop icons for their users - despite the availability of pinned list configurations, desktop shortcuts were a part of their everyday workflow.

Oh, by the way I’m Dustin Gullett – I work closely with Steve on everything Intune and Entra related.

### **The Problem**

The issue was first reported when I created a multi-app kiosk using an XML on an older version of Windows 11 24H2. After the device completed enrollment, I noticed that desktop shortcuts were still present. All configurations seemed correct, and everything was functioning as expected. However, after the update rings were applied and the device was restarted, the desktop icons disappeared.

Despite thorough checks, I couldn't find any official documentation from Microsoft, but multiple forums suggested that a cumulative patch in August 2024 had disabled these icons.

### **Initial Troubleshooting Steps**

My first step to troubleshoot was examining the registry value located under HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\Explorer, with the value “NoDesktop" set to 1. This value hides and disables all items on the desktop. My first attempt was to use the Settings Catalog in Intune to disable this setting - however, after multiple syncs, the value remained unchanged, and the desktop icons did not reappear. Manually modifying the registry value using regedit was also unsuccessful due to access restrictions.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1d1b7220-d598-4340-879a-32b14b57c1b5/Picture1.png)

### **A Creative Solution**

I reached out to Jesse, who was experiencing a similar issue. Jesse provided a script that could be run as SYSTEM, which would get the current logged-in user's SID and populate the HKEY\_USERS hive for that user. This script successfully changed the NoDesktop value to 0, restoring the desktop icons. However, the value reverted to 1 after a restart, causing the icons to disappear again.

New-PSDrive -PSProvider Registry -Name HKU -Root HKEY\_USERS -ErrorAction SilentlyContinue
$userName = (Get-WmiObject -Class Win32\_ComputerSystem | Select-Object -ExpandProperty UserName)
$SID = (New-Object System.Security.Principal.NTAccount($userName)).Translate(\[System.Security.Principal.SecurityIdentifier\]).Value
 reg.exe add "HKU\\$SID\\Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\Explorer" /v "NoDesktop" /t REG\_DWORD /d 0 /f | Out-Host
 

### **The Final Fix**

To address this, I modified Jesse's script to create a .ps1 file that would set the NoDesktop value to 0 and run as a scheduled task at login. The script creates a desktop\_fix.ps1 file located under C:\\Programdata\\IntuneScripts and registers it as a scheduled task to run at each logon. This solution ensured that the desktop icons were consistently restored after each login.

$transcriptPath = "$env:ProgramData\\Microsoft\\IntuneManagementExtension\\Logs\\enable\_desktop.log"
Start-Transcript -Path $transcriptPath -Append

#  create content for desktop\_fix.ps1
$script = @'
New-PSDrive -PSProvider Registry -Name HKU -Root HKEY\_USERS -ErrorAction SilentlyContinue
$userName = (Get-WmiObject -Class Win32\_ComputerSystem | Select-Object -ExpandProperty UserName)
$SID = (New-Object System.Security.Principal.NTAccount($userName)).Translate(\[System.Security.Principal.SecurityIdentifier\]).Value
 
reg.exe add "HKU\\$SID\\Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\Explorer" /v "NoDesktop" /t REG\_DWORD /d 0 /f | Out-Host

# Restart explorer.exe to show the desktop
Stop-Process -Name explorer -Force

'@

# Create desktop\_fix.ps1 in C:\\ProgramData\\IntuneScripts
$path = $(Join-Path $env:ProgramData IntuneScripts)
if (!(Test-Path $path)) {
    New-Item -Path $path -ItemType Directory -Force -Confirm:$false
}
Out-File -FilePath $(Join-Path $env:ProgramData IntuneScripts\\desktop\_fix.ps1) -Encoding unicode -Force -InputObject $script -Confirm:$false

# Register the script as a scheduled task to run at each logon
$Time = New-ScheduledTaskTrigger -AtLogOn
$User = "SYSTEM"
$Action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-ex bypass -file \`"C:\\ProgramData\\IntuneScripts\\desktop\_fix.ps1\`""
Register-ScheduledTask -TaskName "EnableDesktop" -Trigger $Time -User $User -Action $Action -Force

# Start the scheduled task
Start-ScheduledTask -TaskName "EnableDesktop"

Stop-Transcript

After deploying this script to the assigned access (kiosk) devices, the desktop icons were successfully restored, and the issue was resolved.
