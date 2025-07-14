---
author: GuestUser
date: Fri, 19 Jan 2024 19:58:28 +0000
description: 'Hey everyone - I figured it was about time I come back and contribute
  some written content after about 3 years (my lord).Yes, I still work with very closely
  with Steve – since I finally showed myself in one of his latest videos, I figured
  I would come'
slug: custom-intune-reporting
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/custom-intune-reporting-PVN2O_thumbnail.jpg
title: Custom Intune Reporting
---

Hey everyone - I figured it was about time I come back and contribute some written content after about 3 years (my lord).

Yes, I still work with very closely with Steve – since I finally showed myself in one of his latest videos, I figured I would come back with a hefty PowerShell script that should help you in some of your management/reporting needs. I will do my best to post more regularly as we keep on keeping in the modern management world.

## Windows Reporting Script
---

To summarize, this is a custom reporting script that I’ve used with a number of my customers. It has changed and evolved a bit over time… It has been particularly helpful when tracking changes around co-management workloads, or onboarding to Windows Updates for Business policies; it also combines some other data that normally isn’t visible in Intune’s device list (group memberships, logged on users, and much more). While there are other solutions in the wild between scripting, log analytics and even Power BI, I wanted something I could run locally with just PowerShell (and maybe organize in excel).

A big thank you to my colleague Logan Lautt for helping with the feature update exporting and several other functions/graph calls/syntax corrections throughout the script.

## Step 1: Application Registration
---

In order to retrieve the necessary data, the script will require an app registration with the following application-based permissions:

-   `Device.Read.All`
    
-   `DeviceManagementConfiguration.ReadWrite.All`
    
-   `DeviceManagementManagedDevices.Read.All`
    
-   `DeviceManagementServiceConfig.Read.All`
    
-   `Directory.Read.All`
    
-   `Group.Read.All`
    
-   `GroupMember.Read.All`
    

You might be curious as to why the second ReadWrite permission is required… well this is simply due to the POST call utilized for the Feature Update report export (deviceManagement/reports/exportJobs). We need to submit the reporting payload we are requesting from Intune – we can then GET the status of the export job, and once it is ready (not null) we can download and import the data. You can likely write similar code for other built-in reports that have the generate and export functions.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/891dc9a5-c4af-4299-b801-353f20e9e843/starwars.jpg)

Please note – this version of the script uses application-based API permissions. You could alter the script to use delegated permissions if preferred; I did include the replacement commands within the script. I have utilized this previously when I also needed to retrieve BitLocker recovery keys, as that particular call only supports delegated permissions (I did not really want the key at the time, I just wanted to confirm a key was present/uploaded to Entra ID).

Having said that, I am only using the deviceManagement calls for encryption status.

## Step 2: Running the script
---

Before you officially run the script, you are welcome to re-arrange and add/subtract any variables in the **$record** object on line 745 – this will control what columns you end up with in your final export. A lot of device object properties are pre-defined into variables starting on line 385, though some may have been skipped based on typical reporting needs (you can always run a GET to a variable in PowerShell and echo it out to see all captured properties, or refer to the [graph documentation](https://learn.microsoft.com/en-us/graph/api/intune-devices-manageddevice-list?view=graph-rest-1.0&viewFallbackFrom=graph-rest-beta&tabs=http)).

When you first run the script, you will be prompted with two questions – which group of devices do you want to report on, and which Feature Update profile do you want to check device statuses from. Both prompts support a response of “All” – I typically report on a specific device group and enter “All” Feature Update profiles (in case they have varying assignments). If you enter “All” for the device targeting as well, the script will take some time to run depending on the number of devices

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/42995d28-2a33-4fa2-81dc-09cbb53a0b24/Screenshot+2024-01-19+142918.png)

This script is intended to be run locally on an administrator’s machine. Is it possible to run such a script elsewhere in an automated fashion? Sure, you could hardcode the All/targeted responses, maybe create a runbook in Entra ID and have the CSV automatically sent somewhere on whatever schedule. However, since I like to import the CSV into formatted Excel templates, and since the report requests are not always on a schedule, I just run it locally to keep things simple.

Go ahead and run that bad boy, be ready to sit for a few if you have a larger-sized tenant.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/37776884-b6c4-407e-9fcb-71f6d1ef7ed7/caddy.png)

Oh – one thing about these scripts and running through so many devices and calls… if you suddenly start seeing 403:Forbidden for each device, this is likely because the authorization token is no longer valid after a long period of time. I tend to do my testing in larger size tenants – because of this, and because of the 403s I was frequently running into, my current version of the script is extra cautious with re-authenticating via the app registration. I’ve commented on a few sections in the script where I re-run the function, as it may be overkill in your use case (I’m not sure if it’s detrimental to refresh your auth token too quickly).

Related to this, you can modify the graph throttling portion of the script (line 804) to re-authenticate after whatever number of devices seems appropriate (there’s one counter used for pausing for 10 seconds after 1,500 devices – the nested if statement is optional if you want to wait longer to re-authenticate to graph, though I just make it re-auth at 1,500 to be safe):

    if($deviceCount % 1500 -eq 0)
    {
        Write-Output "$deviceCount devices have been processed so far.  Now sleeping
        for 10 seconds to avoid graph limiting..."
        Start-Sleep -Seconds 10

        #if($deviceCount % 3000)
        #{
            Write-Host "Re-authenticating with app registration..."
            $headers = Connect\_To\_Graph
        #}
    }

Another thing – once in a while I’ll see a Gateway Timeout error (504) when I first start the script. If you see this just cancel with Ctrl + C and try to re-run it, as it’s usually a random network connection issue. However, I’ve also seen this message sporadically show during the device encryption and health attestation calls – this is more likely due to the amount of data being requested in the largest-sized tenants. It should automatically retry if that’s the case, but I did repeat my Connect\_To\_Graph function to hopefully avoid this error. It shouldn’t take too long!

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/4d1d50ca-a469-4f0c-9f7e-015d6677caad/waiting.png)

Step 3: Import your CSV!
------------------------

The colors, Duke, the COLORS! The best part now is being able to sort, filter, and conditionally format (color) different pieces of information. Here is the basic template I use and import to – let’s go through this particular process…

-   After opening the template, I will navigate to File -> Import (Yeah yeah, I’m on a Mac - be quiet Steve). Joking aside, I don’t recall how different the import menu is on Windows, so hopefully it’s about the same for you folks.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/41a8b582-1dc2-46e2-847d-cd2f1ae47f33/Picture1.png)

-   Make sure of course you select CSV file type. On the next screen select Delimited, and star the import on row 2 (NOTE: The file origin will likely have Windows ANSI on a PC by default, which should be fine).
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/95438984-5711-4d08-a092-4122b6c74412/Picture2.png)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/033b49c9-8fbf-471b-b931-fa53be637534/Picture3.png)

For delimiters, we will switch it from Tab to Comma and hit next. For the column data format, I skip this as I have the columns formatted in the xlsx file.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/8a693505-9bb8-4187-a951-bcb9e1409edf/Picture4.png)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ff2c11bf-f82e-4777-a63b-06940b3df856/Picture5.png)

Before I hit Import, I always like to open Properties and uncheck “Adjust column width”. Once I click OK, I can finally click Import.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/504d082e-1dcf-4839-857d-b91539a1a048/Picture6.png)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e6cfbde3-6b6e-4e6a-ad7b-2c291db291a4/Picture7.png)

The Final Outcome
-----------------

50 columns of glorious details… it’s a lot, but it’s all in one place. Again – there’s room for more, and you can certainly take away some columns you don’t care about.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/705a8563-012d-42a6-9cfd-6f0f0440c34a/Picture8.png)

Two things stand out in this script’s current state… some devices were not able to report health attestation details like BitLocker Status and Secure Boot (I even included a redundant TPM Version column, though I would just recommend using the hardwareInformation variable on line 410 instead). This could be from lack of a compliance policy requiring such settings, but I did not get a chance to confirm; you’ll also notice the last two columns on health attestation status are null when BitLocker/Secure Boot are unknown.

Second, the device scope tags. I always forget that the tags are represented by a short number in graph instead of the actual names – at some point I will need to update this script to first gather all tags and convert them to their friendly name.

I have the sample uploaded to the GitHub here: https://github.com/stevecapacity/IntunePowershell/blob/main/intuneGraph\_WinReporting\_Script.ps1

I hope this helps you in some of your device hunting and reviewing environment changes. I may play around and add in a few additional data exports… perhaps some win11 readiness details? stay tuned!
