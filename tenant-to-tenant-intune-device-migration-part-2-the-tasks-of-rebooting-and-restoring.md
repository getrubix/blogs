---
author: steve@getrubix.com
date: Fri, 28 Jul 2023 15:53:01 +0000
description: '"You might not have noticed, but in the last post when we covered the
  StartMigrate.ps1 script, we set a lot of tasks. These are responsible for handling
  our post-migration activities and there is a very specific method to their madness.Task
  FlowFor as elaborate as this task system"'
slug: tenant-to-tenant-intune-device-migration-part-2-the-tasks-of-rebooting-and-restoring
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/tenant-to-tenant-intune-device-migration-part-2-the-tasks-of-rebooting-and-restoring_thumbnail.jpg
title: Tenant to Tenant Intune Device Migration Part 2 The Tasks of Rebooting and
  Restoring
---

You might not have noticed, but in the last post when we covered the **StartMigrate.ps1** script, we set _a lot_ of tasks. These are responsible for handling our post-migration activities and there is a very specific method to their madness.

Task Flow
---------

For as elaborate as this task system is, they follow a very simple flow.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/d53da0b2-7c45-40f4-9137-98bef909a6e5/TaskFlow2.png)

-   The **StartMigrate.ps1** script is deployed in an app package from Intune in Tenant A
    
-   The script then sets a Windows scheduled task for each accompanying XML file in the package
    
-   After the user logs into the device with their Tenant B credentials, each script is run based on it’s trigger settings
    
-   Each task calls a PowerShell script of the same name, which then disables that task after it has run.
    

To ensure we don’t run into permission and privilege issues, each XML task will launch the corresponding PowerShell script as **NT AUTHORITY\\SYSTEM** context.

What is happening?
------------------

Let’s talk about what each task is doing, how it’s doing it, and when it needs to happen. Because there is so much going on in each of the 6 tasks, I’m going to cover 2 at a time so I can be thorough, without writing a novel.

### **MIDDLE BOOT (TRIGGER: ON BOOT)**

At the end of **StartMigrate.ps1**, the PC un-joins from Tenant A Azure AD and reboots. In order to be able to properly log into Tenant B, a subsequent reboot has to happen. In the first iteration of the process, we instructed end users to manually reboot before signing in. I’m sure you can guess how that went…

So the **MiddleBoot** task is set to run on the first boot _after_ the initial reboot. It’s original purpose was just to run the **MiddleBoot.ps1** script to execute a reboot in 30 seconds with a one-liner and then disable the task:

```
shutdown -r -t 30
Disable-ScheduledTask -TaskName "MiddleBoot"
```

Simple and effective. But once we started migrating the user data, we came across an issue. Most of the time, a user’s profile name is something like “First.Last” in both Tenant A _and_ Tenant B. So that would mean once they login after the 2nd reboot with Tenant B credentials, we are looking at a potential issue of duplicate profile names.

We have to rename the Tenant A local profile name before the 2nd reboot. So why not include it in the **MiddleBoot.ps1** script? It seems like a great idea, but how will the script know the name of the Tenant A user profile? Wait a minute- didn’t we capture that in the **StartMigrate.ps1** script? Yes, we did! And luckily, we stored that value in a local XML file called **MEM\_Settings.xml**.

Here is what the finished product looked like:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/02728329-2590-4a71-b253-30d05ecc78bd/middleboot.png)

Note that we’re also creating the **post-migration.log** file to capture all of the output from the task scripts. Subsequent scripts will append their transcripts to that file.

### **RESTORE PROFILE (TRIGGER: 5 MIN AFTER LOGON)**

Alright- after the 2nd reboot, our users can log into the device using Tenant B credentials. If everything worked up to this point, they should be able to access basic apps like Outlook and Teams, signed in to Tenant B and ready to work. Let’s get their data restored.

Before backing up any local data in **StartMigrate.ps1**, we checked to see if there is enough disk space. If there is not, then we skipped the backup. I’ve added a line to create a file, **_MIGRATE.txt_** as a local flag we can use to know whether we need to restore data or not. After all, if we’re not backing up, then we’re not restoring.

If the flag exists, we move on to retrieve all of the paths we backed up. The data paths are stored in our local config file, **_MEM\_Settings.xml_**. We’ll retrieve these paths from the XML file and add them to a new **_$locations_** variable.

Once we have that, the next part is just the inverse of the original data backup from **StartMigrate.ps1**. We iterate through the locations and for each one, construct the public temp path and our current user profile path. Robocopy makes sure the junction directories are skipped and everything is logged.

After the data migration is complete, we reset the policy to show the last signed-in user at the logon screen. I’m going to have the device reboot one more time because of all the data we moved to the user profile, and same as before, this script will disable the task that ran it. Take a look:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/476380e7-c39a-4a61-a0f5-042d9404483d/restore.png)

Next time…
----------

We’re in the thick of it now! The task files above in addition to the updated **StartMigrate.ps1** can be downloaded [here](https://github.com/stevecapacity/IntuneMigration) in my new migration repo.

In the next post we’ll cover the **GroupTag** and **SetPrimaryUser** tasks.
