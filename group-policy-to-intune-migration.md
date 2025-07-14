---
author: steve@getrubix.com
date: Fri, 25 Mar 2022 19:43:34 +0000
description: '"This is a big one, folks. It’s been a long time coming and my whole
  team is legitimately thrilled to have this now. In case you’re not as hyped as me
  yet, here’s some backstory. For Windows devices managed with Intune and joined to
  Azure"'
slug: group-policy-to-intune-migration
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/group-policy-to-intune-migration_thumbnail.jpg
title: Group Policy to Intune Migration
---

This is a big one, folks. It’s been a long time coming and my whole team is legitimately thrilled to have this now.

In case you’re not as hyped as me yet, here’s some backstory. For Windows devices managed with Intune and joined to Azure Active Directory, there is no way to implement existing group policy on the machine. Obviously, this would need to be addressed if there was any hope for PCs to be managed in the cloud. Microsoft’s first stab at this was the [MDM Migration Analysis Tool, or MMAT](https://github.com/WindowsDeviceManagement/MMAT), for short. This tool would run on an existing domain join machine and spit out an excel report of which policies were available in Intune as CSP profiles (modern Intune settings).

After realizing how tedious it was, Microsoft then implemented [Group Policy Analytics](https://docs.microsoft.com/en-us/mem/intune/configuration/group-policy-analytics) in Endpoint Manager. This time, you could export your GPO right from the domain in an XML file and upload this to Intune. You were then presented with an amazing chart that broke down how many of your policies are compatible with Intune and what the path is to set them. While fancy looking, there was just one issue; you could _only_ look.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/09db85ad-b324-419b-a975-d5ba24020641/gpoAnalytics.png)

If you decided that you wanted to migrate 100 of those compatible settings, you would have to go line by line and configure them yourself with either the settings catalog, administrator templates or a custom policy. Depending on how many are required, you’re easily looking at hours, if not days of manual work.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/a9942ca6-30f7-474c-9833-7663d8782566/gpo+settings.png)

The solution
------------

So how exactly are we going to automate this process? Here’s the flow:

-   Use GPO Analytics as you would today to have Intune sort through them. Follow the instructions in the link above.
    
-   Export the GPO Analytics results from Intune to a csv file.
    
-   Run the PowerShell script (which leverages the Microsoft Graph API) we’re about to discuss.
    
-   Enjoy!
    

That’s it! Seems simple enough right? Let’s go into the steps including the permissions needed for the script and how to run it.

Ask for permission
------------------

As with most PowerShell scripts that leverage the Microsoft Graph API, we need the script to authenticate with the appropriate permissions in order for it to do, well, what it needs to do. Create an Azure app registration and make note of the **Client ID** and **Secret** values. If you haven’t done so, please read Mr. Michael Niehaus’s blog on how to do so [here](https://oofhours.com/2019/11/29/app-based-authentication-with-intune/).

Make sure to provide the app registration **Microsoft Graph ->** **Application permissions -> DeviceManagementConfiguration.ReadWrite.All** permission and grant consent.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ce71cd82-43ec-4f48-a5c3-98f0c5d85d4e/api.png)

Two important files
-------------------

Once you’ve ran Group Policy Analytics, go ahead and export the results. You should get a .csv file with all the policy in it. Because of the formatting, you’ll need to switch the headers in the Intune downloaded version to the one in the template I created. You can download it here: [template.csv](https://github.com/groovemaster17/IntunePowershell/blob/main/template.csv).

Replace the first row with the one in my template. Then go ahead and filter out any unsupported or duplicate policy. It’s time to download the second, and most important file, the actual PowerShell script: [gpoToCspMigration.ps1](https://github.com/groovemaster17/IntunePowershell/blob/main/gpoToCspMigration.ps1).

Minor edits
-----------

For the script to work, you have to add your own personal touch to three areas.

First, replace lines **33**, **34** and **35** with your own values from the app registration you created earlier:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/0bb4db79-1471-47e6-8ead-37a6fac1f2b0/script1.png)

Next, set the full path to your .csv file on line **54**:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/467514b8-64ec-43a9-a132-f883b611f7ae/script2.png)

Lastly, choose your own name for the policy on line **195**:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/3a3aa6a7-e400-4d70-b8c6-44d095f7715c/script3.png)

With those edits complete, go ahead and run the script. If all goes well, you should end up with your new policy in Endpoint Manager in **Device -> Configuration Profiles**.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/2ee7dd41-24be-42bc-b5fc-ee2417dcbbb5/success.png)

Just because we can, should we?
-------------------------------

I’ve always preached about modern management and how organizations need to abandon technical debt. This is still my preference and if you can deploy PCs through Intune with security baselines and configuration profiles, then more power to you.

The reason I decided to create this was two fold. First off, there are some situations where the policy is required and manually combing through the corresponding CSPs is just way too time consuming.

Secondly, I often find folks who aren’t sure if they need to keep certain policies or not. In that case, what better way to test then deploy the settings straight from GPO analytics in addition to security baselines and modern policy. Intune will report on conflicts, you can be sure of that. If the policy we applied does indeed conflict, then we know it is no longer needed.

Whichever reason you end up needing to migrate policies over, at least we can finally do more than just look!
