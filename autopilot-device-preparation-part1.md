---
author: steve@getrubix.com
date: Sun, 09 Jun 2024 01:09:40 +0000
description: '"Microsoft has just released its latest flavor of Windows Autopilot
  known as Autopilot Device Preparation, and it’s been getting a lot of press this
  week. Some call it Autopilot V2, some call it “half-baked”, and some have said much
  worse. But regardless, it’s here and we’ve been putting"'
slug: autopilot-device-preparation-part1
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/autopilot-device-preparation-part1_thumbnail.jpg
title: How to configure Autopilot Device Preparation Part 1 Set it up
---

Microsoft has just released its latest flavor of Windows Autopilot known as [Autopilot Device Preparation](https://techcommunity.microsoft.com/t5/intune-customer-success/announcing-new-windows-autopilot-onboarding-experience-for/ba-p/4161000?utm_source=dlvr.it&utm_medium=twitter), and it’s been getting a lot of press this week. Some call it Autopilot V2, some call it “half-baked”, and some have said much worse. But regardless, it’s here and we’ve been putting it through its paces.

> _If you haven’t already, check out my videos on it_ [_here_](https://www.youtube.com/playlist?list=PLKROqDcmQsFkL4uCcCHUMRTdW7IM-IIoq)_._

As a companion to those videos, I’ll be publishing several articles on getting started and setting it up, testing it out, and maybe some troubleshooting. In this first post, we’ll look at getting started and creating our device preparation policy.

Prepare the tenant
------------------

### Allow personal enrollment

The first thing we want to do is make sure we are allowing personal enrollment of Windows devices in Intune, as that is the only way to currently test APV2.

As stated in the Microsoft docs above, we will soon have the ability to upload a PCs manufacturer, model, and serial number to the corporate identifiers list in order to block personal devices again.

> _I’m going to show you how to allow personal devices with the default platform restrictions, but if you’d like, you can follow_ [_this link_](https://learn.microsoft.com/en-us/mem/intune/enrollment/create-device-platform-restrictions) _to create a new platform restrictions policy to test with._

-   Logon to [intune.microsoft.com](http://intune.microsoft.com/) and navigate to **Devices** > **Enrollment** and select **Device platform restriction**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/902ec7ce-3f47-4917-aff6-f168400577b3/Screenshot+2024-06-08+at+2.47.56%E2%80%AFPM.png)

-   Select **Windows restrictions** and then click **All users**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/53cb687d-f0f6-4f27-ba83-091980b51f1d/Screenshot+2024-06-08+at+2.49.06%E2%80%AFPM.png)

-   Click **Properties** and ensure that _Personally owned_ is set to **Allow** for _Windows (MDM)_
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/25fb9a58-5c7c-4a61-8514-84b931a9755e/Screenshot+2024-06-08+at+2.51.59%E2%80%AFPM.png)

### Create a special device group

One of the new features of APV2 deployment profile is that it will place a PC in a device group for us during provisioning. The reason I call it a ‘special’ group is because it must adhere to the following:

-   Be a _Security_ group
    
-   Have the _Membership type_ set to **Assigned**
    
-   Be owned by the **Intune Autopilot ConfidentialClient** service principal
    

The first two are no problem, but what is that last thing? Don’t worry; I’ll explain.

-   To create the group in Intune, navigate to **Groups** > **All groups** and click **New group**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/8caaf7dd-87bc-4b44-83e0-218a6bafaa55/Screenshot+2024-06-08+at+2.58.27%E2%80%AFPM.png)

-   Fill out the following fields:
    
    -   _Group type_: Choose **Security** from the drop down
        
    -   _Group name_: This one is obvious; name the group what you’d like
        
    -   _Group description_: See above
        
    -   _Membership type_: Assigned
        
-   For _Owners_, click **No owners selected**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/9a956b66-2edf-482e-af27-5e4816c99e31/Screenshot+2024-06-08+at+2.59.51%E2%80%AFPM.png)

-   On the “Add owners” page, search for _Intune_ and select **Intune Autopilot ConfidentialClient** and then click **Select** and then create the group
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ab339ad7-754c-4b6f-8173-a56aed246314/Screenshot+2024-06-08+at+3.01.17%E2%80%AFPM.png)

> **_IMPORTANT_**_: Make sure the service principal ID for the owner is_ **_f1346770-5b25-470b-88bd-d5744ab7952c_**_. It may also have a display name of_ **_Intune Provisioning Client_**_, but if the ID is the same then it doesn’t matter. If for some reason the ID is not present at all, you will have to manually add it. I will guide you through that in the next section._

### Add the client manually (only if required)

When you create your group based on the previous section, and you try to add the **Intune Autopilot Confidental/Provisioning** client as the owner, you may find it does not exist.

If that’s the case, perform the following:

-   Launch PowerShell and install the Azure AD module with the following command
    
    -   `install-module azuread -confirm:$false -force`
        
-   Import the module
    
    -   `import-module azuread`
        
-   Connect to the Microsoft Graph and authenticate with Global Administrator or a role that has permissions to add service principals
    
    -   `Connect-AzureAD`
        
-   After you’ve signed in, run the following command to add the service principal
    
    -   `New-AzureADServicePrincipal -AppId f1346770-5b25-470b-88bd-d5744ab7952c`
        

Now repeat the steps in the previous section to add the client as the owner to your group. Remember, it may display as **Intune Provisioning Client** or **Intune Autopilot ConfidentialClient**; either way, it should have the **f1346770-5b25-470b-88bd-d5744ab7952c** ID so you’ll be fine.

Configure device preparation
----------------------------

Now that the tenant is ready, we can configure our device preparation policy. We’ll walk through each phase of the policy.

-   While still logged into Intune, navigate to **Devices** > **Enrollment** and select **Device preparation policies**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/3916f176-4279-4049-8193-e23e30ecf762/Screenshot+2024-06-08+at+2.54.08%E2%80%AFPM.png)

-   Click **\+ Create**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/155627a3-a332-4919-b2c0-6263e6b164a9/Screenshot+2024-06-08+at+2.55.01%E2%80%AFPM.png)

### Introduction

The first section, _Introduction_ gives you a brief overview of the workflow.

-   Click **Next**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/677ce45c-1938-4014-a4aa-fa01b52af931/Screenshot+2024-06-08+at+2.55.50%E2%80%AFPM.png)

### Basics

In _Basics_, fill out the following fields:

-   _Name_: Once again, as easy as it gets; name your policy
    
-   _Description_: I’ll let you figure this one out too
    
-   Click **Next**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/271e0fe6-6ee5-4ba0-b4a8-5ce4d0cc7ffb/Screenshot+2024-06-08+at+2.56.40%E2%80%AFPM.png)

### Device group

The group you choose here is the group that we created in the previous section. Once the user signs into the PC, Autopilot will place the device in this group.

-   In the search box, enter the name of the special group you created.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/58b4b0e6-4b74-42a8-8bdc-c098aab1e25c/Screenshot+2024-06-08+at+3.03.15%E2%80%AFPM.png)

-   Select your group when it’s displayed and click **Next**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/bf37c7c1-4b57-41c3-bb66-828e6c6d6434/Screenshot+2024-06-08+at+3.03.52%E2%80%AFPM.png)

### Configuration settings

The configuration settings are the real bulk of the policy. There are four sections: _Apps_, _Scripts_, _Deployment settings_, and _Out-of-box experience settings_.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/0206ee74-8606-441c-8841-a9f3d557c19f/Screenshot+2024-06-08+at+3.04.59%E2%80%AFPM.png)

**Apps**

-   Click **\+ Add** in the _Apps_ section to bring up the _Select Apps_ menu
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/05431f40-3a97-4a4d-a89b-984eabcf2012/Screenshot+2024-06-08+at+3.05.16%E2%80%AFPM.png)

> _This is similar to the ESP (Enrollment Status Page) from Autopilot V1. Here, you can choose up to 10 applications that will be monitored for installation during the provisioning. Keep in mind, in order for the apps to be deployed, they must be assigned to the device group you created in the previous section._

-   Click **\+ Add** next to each app you want to add. You will see them populate below under _Selected Apps_
    

-   When finished, click **Save**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/c5653c31-5033-43a0-adaa-811ca311bd67/Screenshot+2024-06-08+at+3.06.21%E2%80%AFPM.png)

**Out-of-box experience settings**

-   Complete the following fields and options:
    
    -   _Minutes allowed before showing installation error_: Pretty self-explanatory; basically how much time you should allow for your apps and scripts to finish provisioning before displaying an error to the user
        
    -   _Custom error message_: Here you can write your own message the end user will see in the event of a provisioning failure
        
    -   _Allow user to skip setup after multiple attempts_: If the provisioning repeatedly fails, the end user will have the option to proceed to the desktop. Be aware that this means the user may be using the PC without critical applications or settings present.
        
    -   _Show link to diagnostics_: If the provisioning fails, a link will be displayed so the end user can click on it to generate logs needed to troubleshoot the issues.
        

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/2262d770-c949-4c9d-ae97-ce3863efd13b/Screenshot+2024-06-08+at+3.07.28%E2%80%AFPM.png)

**Scripts**

-   Click **\+ Add** in the _Scripts_ section to bring up the _Select Scripts_ menu
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/55d6c14b-c587-4776-87a5-f1b9740f62ed/Screenshot+2024-06-08+at+3.08.25%E2%80%AFPM.png)

> _This part is completely new to Autopilot, as previously any PowerShell scripts assigned to the device would be enforced during provisioning. Now, just like the Apps, you can choose which scripts will take place during Autopilot provisioning if they’re assign to your device group._

-   Click **\+ Add** next to each script you want to add. You will see them populate below under _Selected Scripts_
    
-   When finished, click **Save**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/6a610226-8e40-4564-8ed9-e3efc9741c64/Screenshot+2024-06-08+at+3.09.19%E2%80%AFPM.png)

**Deployment settings**

-   The settings _Deployment mode_, _Deployment type_, and _Join type_ all currently have one value that cannot be changed. APV2 is designed for a single-user driven Entra ID (Azure AD) joined PC.
    
-   _User account type_: Choose **Standard User** or **Administrator** depending on what privilege you want to provide to your end user upon completing the provisioning
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/84af5e43-7b54-4e29-9d26-d4c9940769e1/Screenshot+2024-06-08+at+3.10.30%E2%80%AFPM.png)

### Scope tags

Here, you can apply any scope tags you utilize in your tenant for this deployment profile.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/455a2bf0-38e2-41ab-b8e4-d72d605def83/Screenshot+2024-06-08+at+3.11.17%E2%80%AFPM.png)

### Assignments

These settings for APV2 are designed to be user targeted. In this section, you will select the user group/groups that will receive the device preparation policy.

-   In the search box, enter the name of the user group you’d like to use
    
-   Select your group when it’s displayed and click **Next**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/fa3d8bb3-8ba8-4932-aa17-023b76dbe2f2/Screenshot+2024-06-08+at+3.12.59%E2%80%AFPM.png)

### Review + create

Take a moment and review the settings you’ve configured for each phase.

-   When ready, click **Save**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ba2c5d95-a241-440d-b2e6-76a2c6d7f120/Screenshot+2024-06-08+at+3.14.10%E2%80%AFPM.png)

Done for now
------------

Good job! You’ve set up Autopilot device preparation in your tenant. In part 2, we’ll look at how to setup physical PCs and virtual machines for testing and walk through the enrollment process.
