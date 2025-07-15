---
author: GuestUser
categories:
- intune
- powershell
date: Thu, 19 Sep 2024 18:38:34 +0000
description: 'A while back Steve posted a video on removing the per-user installs of Teams Classic (which also includes some odd rant about communal food). While this script still generally works, there is a newer option available using the bootstrapper exe.'
slug: bootstrap-the-classic-away-a415x
tags:
- intune
- script
- powershell
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/bootstrap-the-classic-away-a415x_thumbnail.jpg
title: Bootstrap the Classic Away
---

A while back Steve posted [a video](https://youtu.be/Zmk44EuMud4?si=zTyMrMrcsOhFwPC6) on removing the per-user installs of Teams Classic (which also includes some odd rant about communal food). While this script still generally works, there is a newer option available using the bootstrapper exe.

## The Download script
---

The latest versions of teamsbootstrapper.exe will remove the machine-wide installer and per-user instances of Teams Classic. Since Microsoft [documents](https://learn.microsoft.com/en-us/microsoftteams/new-teams-bulk-install-client#remove-classic-teams-for-all-users) the minimum bootstrapper version as 1.0.2414501, I figure it would be helpful to build a script that downloads the latest version each time I want to run it:

```pwsh
#Download bootstrapper
$bootstrapper = "C:\\Windows\\Temp\\teamsbootstrapper.exe"
Invoke-WebRequest "https://go.microsoft.com/fwlink/?linkid=2243204&clcid=0x409" -OutFile $bootstrapper
Start-Sleep -Seconds 3

#Run it
Start-Process $bootstrapper -ArgumentList '-u' -Wait
```

## Did it work?
---

At first, I thought this wasn’t working, as I could still see the machine-wide installer in control panel, and I could also still see the user app in the start menu. However, I forgot about the fact that the machine-wide installer doesn’t always show correctly in control panel (in fact, it often doesn’t show in the first place when installed as System – for me it was showing since I installed it earlier as my local user account, and the entry remained after I ran the bootstrapper cleanup as System).

Once I cleared everything, re-installed the machine-wide as System, and confirmed it didn’t show in control panel, I ran the bootstrapper and saw that `C:\Program Files (x86)\Teams Installer` was in fact removed. I also verified the registry key for the MSI guid was originally present during install, and removed after the bootstrapper cleanup. That path btw is located in:

`HKLM\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\\Uninstall`

## Intune fix
---

Also, with the user app itself, Teams Classic was still present **until I restarted the machine** – once I logged back in, it was completely gone. SO – how should we format this for Intune?

We could just do a platform script… but I would like to 
1. Track the deployment of this script, and... 
2. Notify the user that a reboot needs to be performed when the script is complete. I was debating coming up with a fancy detection script to check that the program files folder and the user-level apps were deleted, but I opted for a simple log file instead.

Here’s what I added into the main script:

```pwsh
$logPath = "C:\\ProgramData\\YourOrg"
$logFile = "teams\_classic\_removal.txt"

if(!(Test-Path $logPath))
{
    mkdir $logPath
}

$flag = "Attempted Teams classic removal via teamsbootstrapper.exe"
New-Item -ItemType File -Path $logPath -Name $logFile -Force
Add-Content "$logPath\\$logFile" $flag | Set-Content "$logPath\\$logFile" -Force

#Cleanup
Remove-Item -Path $bootstrapper -Force
```

Now that we have our script, we can configure the .intunewin package as follows:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/dc419dd5-ce3a-4878-90ea-8e3ce1fd6209/Picture1.png)

- `Install command:` %windir%\\sysnative\\WindowsPowerShell\\v1.0\\powershell.exe -noprofile -executionpolicy bypass -file .\\teamsClassic\_BootstrapperRemoval.ps1  

- `Uninstall command:` cmd /c del "C:\\ProgramData\\YourOrg\\teams\_classic\_removal.txt"

- `Installation time required (min):` 15

- `Allow available uninstall:` Yes

- `Install behavior:` System

- `Device restart behavior:` Determine behavior based on return codes

For the Return codes, you might have noticed I changed the `0` from the default Success to Soft reboot – this is simply because I forgot to add an exit code of `3010` at the end of the script, which would have accomplished the same thing. Either way, this will notify the end-user that a reboot is required for the installation to complete.

Also, since our detection rule is looking for the presence of `C:\\ProgramData\\YourOrg\\teams\_classic\_removal.txt`, we can specify deleting the file as the “uninstall” command for the package (this won’t put Teams Classic back; it will simply let us re-run the removal again if we need).

And that’s it. The only other thing to note – Microsoft does state that after installing the new Teams client and setting the **New Teams Only** policy (or reaching the end of life period for classic Teams), they will attempt to remove the classic version after a period of time. There’s some [scenarios and uncertainties](https://learn.microsoft.com/en-us/microsoftteams/teams-classic-client-uninstall) they call out, so hopefully this script will help for the time being.
