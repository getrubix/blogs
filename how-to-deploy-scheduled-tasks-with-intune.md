---
author: steve@getrubix.com
date: Mon, 25 Nov 2024 21:31:47 +0000
description: '"We’re all sick of scheduled tasks, but let’s face it; you probably
  still need to deploy them on your Windows machines. And now that you’re invested
  in the brave, new world of Intune, you’re going to need a way to deploy them.This
  guide will serve as a written"'
slug: how-to-deploy-scheduled-tasks-with-intune
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/how-to-deploy-scheduled-tasks-with-intune_thumbnail.jpg
title: How to Deploy Scheduled Tasks with Intune
---

We’re all sick of scheduled tasks, but let’s face it; you probably still need to deploy them on your Windows machines. And now that you’re invested in the brave, new world of Intune, you’re going to need a way to deploy them.

This guide will serve as a written companion to my recent video, _“How to Deploy a Scheduled Task with Intune”_. You can watch it [here](https://youtu.be/W-qooDTldZI).

What Are Scheduled Tasks?
-------------------------

Scheduled tasks execute commands triggered by a specific event, whether it’s running a PowerShell script or performing system checks. When configured correctly, they save you time, effort, and frustration, leaving you free to focus on the bigger picture.

You have two options for deploying tasks through Intune.

Method 1: The PowerShell and Packaging Way
------------------------------------------

### Step 1: Use Task Scheduler to Build the Blueprint

![SCR-20241124-qlob.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569490929-28RNRPD8DHR6430NEK78/SCR-20241124-qlob.png)

![SCR-20241124-qlob.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569490929-28RNRPD8DHR6430NEK78/SCR-20241124-qlob.png)

![SCR-20241124-qlrl.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569490944-RKKRF3DG94M6GLYNLSOH/SCR-20241124-qlrl.png)

![SCR-20241124-qlrl.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569490944-RKKRF3DG94M6GLYNLSOH/SCR-20241124-qlrl.png)

![SCR-20241124-qmcw.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569491516-ACSLQNAL4G3QHGK1Q8X8/SCR-20241124-qmcw.png)

![SCR-20241124-qmcw.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569491516-ACSLQNAL4G3QHGK1Q8X8/SCR-20241124-qmcw.png)

![SCR-20241124-qmhr.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569491711-1O9WEBZ7KL6SVH1NTRLD/SCR-20241124-qmhr.png)

![SCR-20241124-qmhr.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569491711-1O9WEBZ7KL6SVH1NTRLD/SCR-20241124-qmhr.png)

![SCR-20241124-qmoc.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569492284-K8GKACRP2FCG8FJQ3PQJ/SCR-20241124-qmoc.png)

![SCR-20241124-qmoc.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569492284-K8GKACRP2FCG8FJQ3PQJ/SCR-20241124-qmoc.png)

![SCR-20241124-qmva.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569492714-090JYHMOLFYDB84AWB54/SCR-20241124-qmva.png)

![SCR-20241124-qmva.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569492714-090JYHMOLFYDB84AWB54/SCR-20241124-qmva.png)

![SCR-20241124-qlob.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569490929-28RNRPD8DHR6430NEK78/SCR-20241124-qlob.png) ![SCR-20241124-qlrl.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569490944-RKKRF3DG94M6GLYNLSOH/SCR-20241124-qlrl.png) ![SCR-20241124-qmcw.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569491516-ACSLQNAL4G3QHGK1Q8X8/SCR-20241124-qmcw.png) ![SCR-20241124-qmhr.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569491711-1O9WEBZ7KL6SVH1NTRLD/SCR-20241124-qmhr.png) ![SCR-20241124-qmoc.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569492284-K8GKACRP2FCG8FJQ3PQJ/SCR-20241124-qmoc.png) ![SCR-20241124-qmva.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1732569492714-090JYHMOLFYDB84AWB54/SCR-20241124-qmva.png)

-   Launch **Task Scheduler** and create a new task.
    
    -   Fill in the details:
        
    -   Name: Something descriptive, like “Primary User Check.”
        
-   Security: Set it to **System** with the highest privileges. Your task deserves to run with authority.
    
-   Set **Triggers**:
    
    -   Example: Run at logon, with a one-minute delay for better performance.
        
-   Configure **Actions**:
    
    -   Start a program, such as PowerShell, and include arguments like `-ExecutionPolicy Bypass -File C:\Scripts\UserCheck.ps1`.
        

### Step 2: Export the Task

-   Save the task configuration as an XML file. This will serve as your deployment template.
    

### Step 3: Package Your Task and Scripts

1.  Gather your XML file and any required PowerShell scripts.
    
    1.  _PowerShell script to deploy task XML found in the GitHub link_ [_here_](https://github.com/stevecapacity/IntunePowershell/tree/main/Misc%20Intune/Tasks)_._
        
2.  Use the **Intune Content Prep Tool** to bundle them into a Win32 app.
    

### Step 4: Deploy via Intune

-   Upload your packaged app into Intune, configure detection rules, and assign it to the appropriate devices or users.
    

Method 2: Direct PowerShell Deployment (Quick and Effective)
------------------------------------------------------------

For simpler needs or when you’re short on time, deploy the task directly via a PowerShell script.

### Step 1: Write the PowerShell Script

Create a script that handles everything from task creation to execution. Here’s an example:

```

$TaskAction = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-ExecutionPolicy Bypass -File C:\Scripts\UserCheck.ps1"

$TaskTrigger = New-ScheduledTaskTrigger -AtLogon

$TaskPrincipal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest

Register-ScheduledTask -Action $TaskAction -Trigger $TaskTrigger -Principal $TaskPrincipal -TaskName "Primary User Check"

```

### Step 2: Deploy Through Intune

-   Go to **Devices > Scripts** in Intune and upload your script.
    
-   Ensure you enable the option to run in 64-bit PowerShell. That part matters.
    

### Pro Tips for Tasks

1.  **Choose the Right Method**: Use the packaging method for larger, more complex tasks. Stick with direct scripting for quick setups.
    
2.  **Stay Organized**: Keep scripts and configurations in a central location, like `C:\ProgramData\Scripts`, for easy management.
    
3.  **Test Before Deploying**: Always verify your task on a reference machine before rolling it out across the network.
    

Why This Matters
----------------

At this time, deploying scheduled tasks with Intune is essentially required for medium to large size organizations, especially during the transition to cloud-native Windows management. Once you get the hang of it, they’re actually one of the easier things to deal with in the Intune world.

Good luck!
