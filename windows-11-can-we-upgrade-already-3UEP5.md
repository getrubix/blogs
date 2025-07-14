---
author: GuestUser
date: Thu, 19 Dec 2024 17:06:19 +0000
description: '"I’m back! I’ve been quite busy the last two months getting folks up
  and running with Intune, especially in regards to planning for Windows 11.Since
  the deadline is fast approaching (it’s nearly 2025 already!), I figured I’d outline
  my overall approach to getting ready for Windows 11. You"'
slug: windows-11-can-we-upgrade-already-3UEP5
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/windows-11-can-we-upgrade-already-3UEP5_thumbnail.jpg
title: Windows 11 Can we upgrade already
---

I’m back! I’ve been quite busy the last two months getting folks up and running with Intune, especially in regards to planning for Windows 11.

Since the deadline is fast approaching (it’s nearly 2025 already!), I figured I’d outline my overall approach to getting ready for Windows 11. You might have seen my previous content on reporting and remediations, but I will include some of those recently updated items as well.  

**Readiness**
-------------

First of all, it’s probably wise to review your current reporting for eligibility status with Windows 11. You have multiple places you can review this in:

**Intune: Reports -> Endpoint analytics -> Work from anywhere -> Windows**

-   This checks the essential hardware requirements for the initial release of Windows 11, and it’s the easiest/quickest way to get this data.
    

**Intune: Reports -> Windows update -> Device Readiness and Compatibility Risk Reports**

-   The device readiness report helps you check system, app, or driver risks that may be present per device, and it is specific to the target version of Windows you select. You do have to also pick a scope tag to generate the report; for organizations that have multiple scope tags, you can export the individual reports and combine them in excel.
    
-   The compatibility risk report is a summarized view of the same data above, with device counts for each problematic app or driver. This is also based on the target version of Windows you select, though you do not have to select a scope tag.
    

**Azure: Windows Update for Business Reports**

-   I certainly recommend having this solution configured. When compared to Intune, there are a few more details here with Feature and Quality Update deployment (the data is a bit more timely as well). This also has Windows 11 readiness, as well as delivery optimization statistics across the board (plus you can run your own KQL queries against the log analytics data).
    

**Configuration Manager: Windows 11 upgrade readiness dashboard**

-   For those with MECM, the report combines the checks of minimum hardware requirements and known compatibility risks with applications and drivers. One other nice thing is you can create dynamic collections based on readiness results, which may help with targeting of software upgrades ahead of time.
    

**Group Policy**
----------------

Once you’ve checked your overall hardware requirements and potential app/driver risks, you’ll want to review the current policies you have in place. When testing Intune’s Feature Update and Update Rings for the first time, a lot of folks will miss some old GPOs that conflict with MDM.

And no – setting the policy to make MDM win over GPO does not fix this. **You need to get rid of those ancient GPOs!** 

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e1b038d0-0001-4f93-ad9f-c36f67cd6425/200.gif)

So which GPOs are potentially problematic? If you look at a cloud-native device without GPO, you will typically see that no settings exist under the following registry paths:

**_HKLM\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate  
HKLM\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate\\AU_**

However, there are usually some values there for hybrid joined systems that receive GPO. Also, MECM puts some default entries in that path, though they are ok if you shifted your workloads correctly.

While you should generally review and plan to exclude any policies managing the Windows Updates (and Windows Update for Business) categories, here are the big ones that typically conflict with Intune’s settings (these values will be under _WindowsUpdate_ or _WindowsUpdate\\AU_):

-   **Configure Automatic Updates**
    
    -   You may have this disabled in combination with WSUS settings, or you may have this similarly configured like your Update Rings. Nevertheless, you should fully switch to Intune and remove this policy to avoid conflict or complete blocking of updates.
        
-   **Do not connect to any Windows Update Internet locations**
    
    -   Obviously, this will be an issue. We’re shooting for internet-driven management.
        
-   **Remove access to use all Windows Update features**
    
    -   We want the opposite of this; this way the user can check for updates in the Settings application and let them download and do their thing.
        
-   **Target Feature Update Version**
    
    -   If you currently have a policy that’s setting TargetReleaseVersion to say Windows 10 22H2, that will likely conflict any later version you specify in Intune’s Feature Update policy. It is recommended to remove this setting ahead of any upgrade attempts.  
        

There are other granular controls like including drivers, scheduling/deadline settings, and end-user notifications; they won’t necessarily break your attempt to upgrade, but they certainly may conflict with what you are trying to configure in the Update Rings (and perhaps Settings Catalog, which has a few extras the Rings are missing).

For the MECM devices, as long as you shift the Windows Update workload to Intune, the following registry values are OK :

-   **DisableDualScan**
    
    -   This is the policy “Do not allow update deferral policies to cause scans against Windows Updated”. When Windows Update workload is shifted to Intune, this should automatically be set to 0. Otherwise, your group policy may separately be forcing this to 1.
        
-   **SetPolicyDrivenUpdateSourceForFeatureUpdates**
    
-   **SetPolicyDrivenUpdateSourceForQualityUpdates**
    
-   **SetPolicyDrivenUpdateSourceForDriverUpdates**
    
-   **SetPolicyDrivenUpdateSourceForOtherUpdates**
    
    -   These are part of the policy “Specify source service for specific classes of Windows Updates.” Just like the above, all four values should get set to 0 automatically when the workload is shifted.
        
-   **WUServer**
    
    -   This is the server address for WSUS. Again, this is ok if the workload is shifted to Intune.
        
    -   This is typically paired with the value “UseWUServer”, which is location in the sub-registry key WindowsUpdate\\AU.  
        

**Remediations**
----------------

Alright, so you’ve checked your Group Policy and you’re pretty sure you’ve cleaned it up. Are the settings gone now? Nothing got tattooed, I hope?

Let’s check – we can use one of my previous remediation examples to gather the device’s registry values in both WindowsUpdate and WindowsUpdate\\AU:  

**Remediation 1: WindowsUpdate**

$registry = reg.exe query "HKLM\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate" | ConvertFrom-String | Select P2, P4 | ConvertTo-Csv -NoTypeInformation | Select-Object -Skip 1

$final = \[string\]::join(",",($registry.Split("\`n")))
Write-Host $final

**Remediation 2: Windows Update AU**

$registry = reg.exe query "HKLM\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate\\AU" | ConvertFrom-String | Select P2, P4 | ConvertTo-Csv -NoTypeInformation | Select-Object -Skip 1

$final = \[string\]::join(",",($registry.Split("\`n")))
Write-Host $final

﻿I will admit, it’s not the prettiest presentation, but it’s tough to squeeze that many entries into a single-line string that’s readable in Remediations (I am certainly open to any suggestions to clean up the results). You can export the results from this remediation and do some manual filtering in excel by typing **“valuename”,”value”**. More on this in a few…

**Intune Settings**
-------------------

From a requirements standpoint with Intune, make sure you have the following settings enabled prior to testing your Feature Update and/or Update Ring policies:

-   Telemetry
    
-   Windows Data
    
-   Windows health monitoring
    

Also, if you plan to deploy the Feature Update policy using the gradual rollout function, you will need to enable the policy “Allow WUfB Cloud Processing”. For the Windows Update for Business reports, you can optionally disable “Configure Telemetry Opt In Settings Ux” and “Configure Telemetry Opt In Settings Notification”, and I would certainly recommend setting “Allow device name to be sent in Windows diagnostic data” to Allowed.

Lastly, I want to emphasize that I default to Feature Update policy as the primary method for Windows 11 upgrades. However, keep in mind that you have alternate options available:

-   **Target Feature Update Version** (in the settings catalog)
    
-   **Update Ring: Upgrade Windows 10 devices to Latest Windows 11 release**
    

**NOTE:** If you’re using Update Rings AND Feature Update policy, don’t worry about flipping this policy to Yes.

The Ring alone doesn’t let you specify which flavor of Windows 11, and the Target Version policy doesn’t integrate into the Reports in Intune. However, these options may be helpful if not all requirements can be met (Entra Registered devices would have to utilize these instead of Feature Update, for example).

**Graph Time!**
---------------

Now that Steve has recently reviewed some essentials with Graph API, you’ll have an easier time following this script I’ve used with multiple folks.

I originally did a video with Steve on this script to gather all sorts of data across Intune – it still collects a lot of said data, but the final output is trimmed to what’s relevant for Windows Update. What’s also neat is I included the upgrade eligibility data from Endpoint Analytics.

And - now that we have 1-2 remediations in Intune, we can even pull those results and incorporate it into the report!

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/878a0906-916b-4d48-916e-57b5cae77302/great.gif)

You can download the script by [clicking here](https://github.com/stevecapacity/IntunePowershell/blob/55b00e29e85bb86d8a32d506451f6b24d96a9151/Win11_Scripts/Intune_W11_Reporting_Script.ps1). Feel free to set it to either delegated or application-based permissions – just make sure to select the correct authentication settings in the script around line 54 (and make sure you have separately loaded MSAL.PS once on your computer for delegated-auth scripts).

Also, you will need to update the code where your Remediation statuses are pulled. Simply retrieve the object IDs of the Remediation packages you created in Intune and update the variables on lines 107-108. I could have gotten fancy by looking up the remediation name and converting to object ID, but you can just get the ID from your browser’s URL bar after clicking on the remediation.  

**Excelsior!**
--------------

What would a CSV export be without a beautifully colored Excel file to import to? Grab the template [from here](https://github.com/stevecapacity/IntunePowershell/blob/55b00e29e85bb86d8a32d506451f6b24d96a9151/Win11_Scripts/REPORT_TEMPLATE%20\(1\).xlsx) and follow the steps below to import your CSV via the legacy text method:

1\. Click in the first empty cell (A2).

2\. In the search bar, type “from text” to get the legacy option to show.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/c3917984-196e-4d28-92b8-045e7ec160e3/excel5.png)

3\. Set the file type to Delimited. Then set the import to start at row 2. Check off “My data has headers” and click Next.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/add098b4-fa13-4519-a065-084a455ffc86/excel1.png)

4\. Set your Delimiters to Comma and click Next.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/8e7faa90-0ae7-4376-992d-e0ee7f2caf25/excel2.png)

  
5\. Click Finish on the next menu. BEFORE you click OK, first click on Properties and uncheck “Adjust column width”. Then you can click OK, and OK again on the last screen.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/97488725-132e-4181-a67e-c2f7a5d16a35/excel3.png)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/65b92b8d-c06f-48e7-84c5-145eeebcad6f/excel4.png)

Let’s see what the final output looks like…

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/90d2cd76-200c-4310-aa41-12f6edda1094/excel6.png)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/016142e6-4f20-4f5f-ba7e-6da2a79d9623/excel7.png)

I have additional hardware details around TPM, Secure Boot, Encryption Status, Manufacturer/Model, Chassis Type, and Processor Architecture – you can always remove these from the final output variable in the script, and the Excel template itself. Same thing for Compliance status – however I do include ConfigMgr client and co-management workload statuses:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/cc3de23c-be85-4145-8f73-4b121e5b54ba/excel8.png)

And while it doesn’t matter so much, I include status of MDM wins over GPO (again, doesn’t really help with Windows Update policies). But – take a look at the remediation data!

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/3b8ec8b1-2d1c-4bcf-a557-e4ac070af2a3/excel9.png)

  
Again, it’s not perfect with how I converted it all to string – however when you want to filter and conditionally format potentially bad values, just do a search in the format as “valuename”,”value”

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e1c2bd39-26d7-4e87-ae92-5a7e9e2c0642/excel10.png)

My conditional formatting at the moment just looks for NoAutoUpdate being set to 1 in the AU key, and DisableDualScan being set to 1 in the parent WindowsUpdate key. You can certainly replicate additional rules for some of the other bad values, which I would recommend for the scansource and target version entries.

Additional hardware and Entra ID details are included by default, but you can always customize to exclude some of this data. Anyway, I hope this helps in your planning and testing of upgrades – you could always incorporate additional remediations to check for other known issues, such as checking for rollback registry entries (usually from 3+ failed upgrade attempts), or other unique errors in the update/CBS logs. Happy upgrading!
