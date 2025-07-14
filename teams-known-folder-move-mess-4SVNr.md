---
author: GuestUser
categories:
- intune
- security
- powershell
- azure
date: Fri, 04 Sep 2020 16:03:36 +0000
description: "Endpoint Manager makes it easy to not only deploy the office suite, but to manage policy around the applications as well. There’s often some policies around Outlook and Office in general that are desired, which are available in the ADMX templates. When it comes to OneDrive specifically, most"
slug: teams-known-folder-move-mess-4SVNr
tags:
- endpoint manager
- azure
- security
- script
- powershell
- intune
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/teams-known-folder-move-mess-4SVNr_thumbnail.jpg
title: Teams Known Folder Move Mess
---

Endpoint Manager makes it easy to not only deploy the office suite, but to manage policy around the applications as well. There’s often some policies around Outlook and Office in general that are desired, which are available in the ADMX templates. When it comes to OneDrive specifically, most folks opt to enable Silent Configuration and Known Folder Move, which redirects the Documents, Pictures, and Desktop folders to OneDrive for Business.

Most companies like the idea of having Office ready to go before the end user hits the desktop - therefore, we assign Office to the Autopilot Devices Group and block the user during ESP until the Office suite is installed (along with other security-related apps as well). This also ensures that the Teams machine-wide installation completes before the user account signs in, which means we will see the user-based installation complete and launch the application automatically when we arrive at the Desktop.

Around the same time that Teams completes the user install and generates a desktop shortcut, the OneDrive Known Folder Move policy kicks in and redirects the desktop. The only issue here - and this is especially true with those who are constantly testing and re-enrolling devices via Autopilot - is you may end up with multiple copies of the “Microsoft Teams.lnk” shortcut. Especially if you forget to delete these like I do. Look at this mess:

![Screen Shot 2020-09-02 at 1.47.15 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599163083679-4RTQKRNHDLMB62BWRWS8/Screen+Shot+2020-09-02+at+1.47.15+PM.png)

Since I primarily work with IT folks who are testing this heavily for the first time, I figure it would be nice to provide a way to cleanup the icons during test deployments. I created a  PowerShell script that generates a reoccurring task for the first 10 minutes of deployment, and it essentially looks for any file names containing “Microsoft Teams -“ in the OneDrive folder path. This one may look a bit odd, as my goal was to do everything within one single script . The other option would have been to wrap three separate files into an intunewin package - that probably would be much prettier, but the scripts tend to hit devices quicker and this needs to hit devices immediately after reaching the desktop.

Take a look at the PowerShell Script below:

$code = @'
  \[System.Runtime.InteropServices.DllImport("Shell32.dll")\] 
  private static extern int SHChangeNotify(int eventId, int flags, IntPtr item1, IntPtr item2);

  public static void Refresh()  {
      SHChangeNotify(0x8000000, 0x1000, IntPtr.Zero, IntPtr.Zero);   
  }
'@


#Attempt icon cleanup during first run
$ErrorActionPreference = "SilentlyContinue"

$owner = Get-Process -Name Explorer -IncludeUserName
$user = ($owner.UserName).Split("\\")\[-1\]

$icons = Get-ChildItem -Path "C:\\Users\\$user\\OneDrive - Company Name\\Desktop" -Recurse

foreach($icon in $icons){
    if($icon.Name -like "\*Microsoft Teams -\*"){
        Remove-Item -Path "C:\\Users\\$user\\OneDrive - Company Name\\Desktop\\$icon" -Force
    }
}

Add-Type -MemberDefinition $code -Namespace WinAPI -Name Explorer 
\[WinAPI.Explorer\]::Refresh()

#Create local copy of script for reoccurring task
$script = '

$ErrorActionPreference = "SilentlyContinue"

$owner = Get-Process -Name Explorer -IncludeUserName
$user = ($owner.UserName).Split("\\")\[-1\]

$icons = Get-ChildItem -Path "C:\\Users\\$user\\OneDrive - Company Name\\Desktop" -Recurse

foreach($icon in $icons){
    if($icon.Name -like "\*Microsoft Teams -\*"){
        Remove-Item -Path "C:\\Users\\$user\\OneDrive - Company Name\\Desktop\\$icon" -Force
    }
}


Add-Type -MemberDefinition $code -Namespace WinAPI -Name Explorer 
\[WinAPI.Explorer\]::Refresh()
'

if(!(Test-Path "C:\\Resources")){
    New-Item -ItemType Directory -Path "C:\\" -Name "Resources"
}

#Carefully add the strings from $code and $script into the file, working around the required single quotes
New-Item -ItemType File -Path "C:\\Resources" -Name "teamsCleanup.ps1" -Force
Add-Content "C:\\Resources\\teamsCleanup.ps1" '$code = ' | Set-Content "C:\\Resources\\teamsCleanup.ps1" -Force
Add-Content "C:\\Resources\\teamsCleanup.ps1" "@'" | Set-Content "C:\\Resources\\teamsCleanup.ps1" -Force
Add-Content "C:\\Resources\\teamsCleanup.ps1" '  \[System.Runtime.InteropServices.DllImport("Shell32.dll")\] 
  private static extern int SHChangeNotify(int eventId, int flags, IntPtr item1, IntPtr item2);

  public static void Refresh()  {
      SHChangeNotify(0x8000000, 0x1000, IntPtr.Zero, IntPtr.Zero);    
  }' | Set-Content "C:\\Resources\\teamsCleanup.ps1" -Force
Add-Content "C:\\Resources\\teamsCleanup.ps1" "'@" | Set-Content "C:\\Resources\\teamsCleanup.ps1" -Force
Add-Content "C:\\Resources\\teamsCleanup.ps1" $script | Set-Content "C:\\Resources\\teamsCleanup.ps1" -Force

#Create XML for reoccurring task
$xml = '<?xml version="1.0" encoding="UTF-16"?>
<Task version="1.2" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
  <RegistrationInfo>
    <Date>2020-08-31T17:48:28.2949925</Date>
    <Author>AzureAD\\JesseWeimer</Author>
    <URI>\\Teams Cleanup</URI>
  </RegistrationInfo>
  <Triggers>
    <RegistrationTrigger>
      <Repetition>
        <Interval>PT1M</Interval>
        <Duration>PT10M</Duration>
        <StopAtDurationEnd>false</StopAtDurationEnd>
      </Repetition>
      <Enabled>true</Enabled>
    </RegistrationTrigger>
  </Triggers>
  <Principals>
    <Principal id="Author">
      <UserId>S-1-5-18</UserId>
      <RunLevel>HighestAvailable</RunLevel>
    </Principal>
  </Principals>
  <Settings>
    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
    <DisallowStartIfOnBatteries>false</DisallowStartIfOnBatteries>
    <StopIfGoingOnBatteries>false</StopIfGoingOnBatteries>
    <AllowHardTerminate>true</AllowHardTerminate>
    <StartWhenAvailable>true</StartWhenAvailable>
    <RunOnlyIfNetworkAvailable>false</RunOnlyIfNetworkAvailable>
    <IdleSettings>
      <StopOnIdleEnd>false</StopOnIdleEnd>
      <RestartOnIdle>false</RestartOnIdle>
    </IdleSettings>
    <AllowStartOnDemand>true</AllowStartOnDemand>
    <Enabled>true</Enabled>
    <Hidden>false</Hidden>
    <RunOnlyIfIdle>false</RunOnlyIfIdle>
    <WakeToRun>false</WakeToRun>
    <ExecutionTimeLimit>PT1H</ExecutionTimeLimit>
    <Priority>7</Priority>
    <RestartOnFailure>
      <Interval>PT1M</Interval>
      <Count>3</Count>
    </RestartOnFailure>
  </Settings>
  <Actions Context="Author">
    <Exec>
      <Command>C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe</Command>
      <Arguments>-Executionpolicy Bypass -WindowStyle Hidden -File "C:\\Resources\\teamsCleanup.ps1"</Arguments>
    </Exec>
  </Actions>
</Task>'

New-Item -ItemType File -Path "C:\\Resources" -Name "teamsCleanup.xml" -Force
Add-Content "C:\\Resources\\teamsCleanup.xml" $xml | Set-Content "C:\\Resources\\teamsCleanup.xml" -Force

#Create task in Task Scheduler
Start-Sleep -Seconds 3
schtasks /create /TN "Teams Cleanup" /xml "C:\\Resources\\teamsCleanup.xml" /f
Start-Sleep -Seconds 3

The only thing you will want to update are the four line with the file paths - make sure you put the correct path with the OneDrive folder name.

Basically, I perform an initial delete of the files if they already exist, and I include the Add-Type command to be able to enforce a desktop refresh after the delete. I then create a local PowerShell script in C:\\Resources with the same commands. This is the part that came out ugly, as I had to work around adding the required single quotes into the file - I would imagine there’s a better way to do this.

I then create an XML file that is used to create a task in task scheduler. I really like this approach with managing tasks - I found that the individual PowerShell commands just don’t present all of the properties that I like to set in a task. I normally have to combine a few commands together to get 80% of the available properties. In this case, I created a trigger that launches the script at task creation, and it repeats every minute for 10 minutes, which should cover the window of the Known Folder Move policy change.

In the end, the files are created and the task is imported and initiated. One thing I noticed is the remaining “Microsoft Teams.lnk” may show a blank image, though this fixes itself when you double click the icon. I would say just remove that as well, and change the line to:

```
if($icon.Name -like "*Microsoft Teams*")
```

![Better?](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599235299503-2FP3NUG60YNHEWXTUDNY/edited.png)

Better?

This may require some additional tweaking depending on the way Office and policies are assigned in your environment, but hopefully this helps with your own testing. Enjoy!
