---
author: GuestUser
categories:
- intune
- security
- powershell
- azure
date: Thu, 12 Dec 2019 19:13:00 +0000
description: >
  Well hello there… my name is Jesse Weimer (no relation to Steve). I work directly with
  Steve in the wonderful world that is Windows 10 and Intune. And Azure. And Autopilot
  and Whiteglove. And iOS. And Android Enterprise too, I guess. Steve brought me in on
  this voyage.
slug: scripting-in-intune-2lwp4
tags:
- azure
- security
- script
- powershell
- scripting
- intune
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/scripting-in-intune-2lwp4_thumbnail.jpg
title: Scripting in Intune
---

Well hello there… my name is Jesse Weimer (no relation to Steve). I work directly with Steve in the wonderful world that is Windows 10 and Intune. And Azure. And Autopilot and Whiteglove. And iOS. And Android Enterprise too, I guess. Steve brought me in on this voyage almost two years ago and I’ve learned a lot over time.

I figured after helping him with his part 3 blog entry, I’d sneak in and write something myself. When I first started working in Intune nearly two years ago, there were a good number of endpoint controls that were not available yet in the console (this was partially due to the capabilities of builds 1703 and 1709). Because of that, I needed rely on a good amount of scripting and had to get more comfortable with PowerShell.

I first learned the basics of pulling EXEs from blob storage and installing them that way… then I got into changing various registry settings to enforce additional policy… and as I started to test more and more scripts, I realized that some commands just simply refused to work in Intune when they worked locally without an issue. I ultimately realized that it came down to a few simple facts:

1.  Scripts in Intune always run as an **elevated** process.
    
2.  Scripts in Intune run as **System/NT Authority**, unless you change it to _“Run with logged on user’s credentials”_ (this setting can help with certain dynamic variables, but then we may lose the rights to run other commands in the script).
    
3.  Scripts can be set to run with the 64bit or 32bit extension.
    
4.  The first two facts remain true when embedding an installation script in a win32 package. The scripts however will run in the 32bit context, since the Intune Management Extension is a 32bit process.
    
5.  Scripts in Intune will only deploy **once** – they can not be scheduled (I could write a script that creates a startup bat, but there’s not much control there).
    

Knowing these facts can help to understand why certain scripts may not work in Intune. This can potentially help solve for dynamic variables not working correctly, or help resolve errors where a system exe could not be called (I had to play around with a printer script to get pnputil to work correctly… maybe I’ll divulge in another post).

So what if we need to run the script with standard privileges? What if we need to schedule the commands to run more than once? Schedule is the key word – the answer here is to create a Task in Task Scheduler, and we can accomplish this with a single PowerShell script.

 As an example of something that many customers still rely on, let’s look at a script that creates drive mappings!

#Create drive mapping bat file - Azure AD-joined machines need the FQDN in the path
$scriptText = '@echo off
net use A: \\\\pathWithFQDN.com\\subfolder /persistent:no
net use B: \\\\pathWithFQDN.com\\subfolder /persistent:no
net use C: \\\\pathWithFQDN.com\\subfolder /persistent:no'
 
#Create xml file to create task to run bat
$xmlText = '<?xml version="1.0" encoding="UTF-16"?>
<Task version="1.2" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
  <RegistrationInfo>
    <Date>2018-08-14T16:26:01.2477131</Date>
    <Author>Jesse Weimer</Author>
    <Description>Map Share Drives</Description>
    <URI>\\Map Share Drives</URI>
  </RegistrationInfo>
  <Triggers />
  <Principals>
    <Principal id="Author">
      INGESTSID
      <LogonType>InteractiveToken</LogonType>
      <RunLevel>LeastPrivilege</RunLevel>
    </Principal>
  </Principals>
  <Settings>
    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
    <DisallowStartIfOnBatteries>false</DisallowStartIfOnBatteries>
    <StopIfGoingOnBatteries>false</StopIfGoingOnBatteries>
    <AllowHardTerminate>true</AllowHardTerminate>
    <StartWhenAvailable>false</StartWhenAvailable>
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
    <Priority>4</Priority>
  </Settings>
  <Actions Context="Author">
    <Exec>
      <Command>"C:\\Windows\\system32\\WindowsPowerShell\\v1.0\\powershell.exe"</Command>
      <Arguments>-ExecutionPolicy Bypass -Command "C:\\Resources\\MapDrives.bat"</Arguments>
    </Exec>
  </Actions>
</Task>'
 
if(!(Test-Path "C:\\Resources")){
    New-Item -ItemType Directory -Path "C:\\" -Name "Resources"
}
 
#Create the script and XML files from the content above
New-Item -ItemType File -Path "C:\\Resources" -Name "MapDrives.bat" -Force
Add-Content "C:\\Resources\\MapDrives.bat" $scriptText | Set-Content "C:\\Resources\\MapDrives.bat" -Force
 
New-Item -ItemType File -Path "C:\\Resources" -Name "mapDrives.xml" -Force
Add-Content "C:\\Resources\\mapDrives.xml" $xmlText | Set-Content "C:\\Resources\\mapDrives.xml" -Force
 
#Get signed-in user's SID to ingest into XML file (this way the task is registered/ran as the user)
$user = Get-WmiObject Win32\_Process -Filter "Name='explorer.exe'" | ForEach-Object    | Select-Object -Unique -Expand User
$domain = Get-WmiObject Win32\_Process -Filter "Name='explorer.exe'" | ForEach-Object    | Select-Object -Unique -Expand domain
$username = $domain+"\\"+$user
$objUser = New-Object System.Security.Principal.NTAccount("$username")
$strSID = $objUser.Translate(\[System.Security.Principal.SecurityIdentifier\])
$userSID = $strSID.Value
$newargs = "<UserId>$userSID</UserId>"
 
#Ingest the SID into the XML
Start-Sleep -Seconds 3
(Get-Content "C:\\Resources\\mapDrives.xml") | Foreach {$\_ -replace "INGESTSID", "$newargs"} | Set-Content "C:\\Resources\\mapDrives.xml" -force
 
#Create the task
Start-Sleep -Seconds 3
schtasks /create /TN "Map Share Drives" /xml "C:\\Resources\\mapDrives.xml" /f
 
#Run the task
Start-Sleep -Seconds 3
schtasks /run /tn "Map Share Drives"

What the hell is all this? It’s actually not that bad – here are the steps performed in the master script:

1.  Define the contents of the script file that will eventually run in Task Scheduler.
    
2.  Define the contents of the XML file that will be used to create the task in Task Scheduler.
    
3.  Create the script and XML files based on the content from above.
    
4.  Modify the new XML file to ensure it contains the SID of the current logged-on user (this is necessary when importing tasks that will run in the user context). Steve helped me with this part.
    
5.  Create the task using the XML file.
    
6.  Run the task (or you could make a shortcut to invoke the task on-demand).
    

In a nutshell, it’s a script inside a task inside a script. The beauty of it is that the XML data contains all settings we needed, including the run level, triggers (if needed), actions, and schedule. Since this is all one script, you can directly upload it to Intune and assign to the desired groups. While I mainly use this format to create drive mappings, I have used this with finicky application installations as well (I have wrapped and deployed some reeaaally old IBM applications with this).

Anyway, hope that helps you in deploying some scripts and maybe some applications. Maybe I’ll post something again, if Steve doesn’t mind. Later!
