---
author: steve@getrubix.com
date: Tue, 01 Aug 2023 12:45:54 +0000
description: '"So, remember how I promised I would talk about the SetPrimaryUser task
  in this post? It’s not that I don’t want to, but I just realized it’s not going
  to make much sense without context. Allow me to introduce you to the two components
  that make"'
slug: tenant-to-tenant-intune-device-migration-part-4-the-bulk-token
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/tenant-to-tenant-intune-device-migration-part-4-the-bulk-token_thumbnail.jpg
title: Tenant to Tenant Intune Device Migration Part 4 The Bulk Token
---

So, remember how I promised I would talk about the **SetPrimaryUser** task in this post? It’s not that I don’t want to, but I just realized it’s not going to make much sense without context. Allow me to introduce you to the two components that make this process possible; the bulk primary refresh token and Windows runtime provisioning package.

I’m going to talk a little bit about each and how they work in the migration process.

Bulk Primary Refresh Token
--------------------------

Since it’s introduction, Windows 10 allowed for Azure AD join, connecting the PC to an Azure tenant without a local Active Directory domain join. Automatic Intune enrollment can also be enabled, allowing a device to enroll as soon as it joins Azure AD.

Typically, this process is facilitated by the end user. Whether during the out-of-box-experience, with Autopilot, or just adding a work/school account in settings, users could join Azure AD and enroll to Intune by signing in with their credentials and receiving a primary refresh token (PRT). The PRT is the primary authentication component for Azure AD. But what happens when an organization wants to enroll a lot of devices very quickly, without the end user?

Enter the PRT’s cousin; the bulk primary refresh token (BPRT). And it does exactly what you think it would. The BPRT acts as a service principle in Azure AD and enables organizations to authenticate PCs directly to Azure AD without the need to sign in over and over again.

While the BPRT was designed for Intune enrollment, there’s a lot more to it. I implore you to check out this ridiculously awesome, [deep-dive](https://aadinternals.com/post/bprt/#:~:text=BPRT%20token%20is%20a%20Bulk,Microsoft%20Endpoint%20Manager%20\(Intune\).) on the subject by Microsoft MVP, [**_Dr. Nestori Syynimaa_**](https://www.linkedin.com/in/nestori/).

Runtime Provisioning Package
----------------------------

A provisioning package is essentially a group of settings and customizations that are bundled into a package that Windows can read. This package is used to automatically apply configurations to a PC, or in most cases, many PCs. The runtime provisioning package is the vehicle used to deliver the BPRT to a device.

In order to create a provisioning package and BPRT, we need the Windows Configuration Designer app. I will tell you, this app has always been somewhat unreliable in the way it works, so I really only use it for the BPRT and least amount of settings possible. I don’t recommend creating full-on device customizations with it. That’s what Intune is for.

Why do we need it?
------------------

When our migration process runs, it will remove the device from Tenant A Azure AD and will need to be moved to Tenant B. But how will the device know what Tenant B is without any user input? That is exactly what the provisioning package will do.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e5f82e46-915f-4f61-aa16-5dc02d85f3d9/Screenshot+2023-08-01+at+8.29.42+AM.png)

By installing the package with our **StartMigrate** script, we’re placing the device in Tenant B before the reboot, but without any user affinity. After the reboot, the user can then sign into Tenant B with the device already being enrolled.

How to make it?
---------------

There are two flavors of the Configuration Designer; one available from the Microsoft Store as a UWP, and the other is included in the Windows Assessment and Deployment Kit (ADK). We’re going to walk through two methods of creating a provisioning package with a BPRT.

### MICROSOFT STORE VERSION

First, download the Windows Configuration designer from the Microsoft Store.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7e6d25b4-51cc-4529-b012-898b526e57ab/blog3-2.png)

Launch the app, and choose the option for “Provision desktop devices” and provide a package name. Our script templates assume the package name is **_migration_**.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/d9e647fd-edf9-46a2-ba05-591658e3a3b1/Screenshot+2023-07-31+093842.png)

Now we will go through the guided setup, configuring very few things along the way. Starting on the “Set up device” page, go ahead and provide a naming schema for the PC. Remember, this is for the PC joining Tenant B, so make sure you choose a name accordingly (no different then choosing a naming convention in Autopilot).

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/0a6fccbb-cb88-4919-8f8c-65e5561ec06d/Screenshot+2023-07-31+093920.png)

On the “Set up network” page, turn off the **Connect devices to a Wi-Fi network** option. We don’t need that. Click Next.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/401d5952-10ec-4a82-8c5d-4dcebb5e27d7/Screenshot+2023-07-31+093934.png)

This is the important step. When you get to “Account Management”, select **Enroll in Azure AD**.

Set **Refresh AAD credentials** to “Yes”.

The standard time for the token expiration is 6 months, so leave that alone.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/f6aeb3df-1ccf-44fb-baf5-75b80ee963f3/Screenshot+2023-07-31+094019.png)

Click on the **Get Bulk Token** button. You will be prompted to sign into Azure AD and I recommend you do so with Global Administrator privileges.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/6f07f31a-6ea7-4cc8-88ca-0712758bbadd/Screenshot+2023-07-31+094042.png)

When asked to **Stay signed in to all your apps**, uncheck the box and click on **No, sign in to this app only**.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/9f333239-a01c-4d15-b441-31c60c840ecd/Screenshot+2023-07-31+094100.png)

If successful, you should see that the bulk token was fetched successfully from Azure AD.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/07cb67a4-e3d8-493b-a3e4-bf7480fc4d14/Screenshot+2023-07-31+094117.png)

Before clicking next on this page, go ahead and set a local admin account. This will give us a way to access the device locally should we encounter an issue with the migration process.

Click next through the **Add applications** and **Add certificates** pages. When you get to the **Finish** page, click on “Create”.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/366455fe-56e9-4b80-9ad4-e26bcfc1593c/Screenshot+2023-07-31+094133.png)

Your provisioning package is now available in the _%USERPROFILE%\\Documents\\Windows Imaging and Configuration Designer (WICD)_ directory. There are several other files generated with the package, including a “customizations.xml” file that is a manifest of everything we configured while going through the guided setup.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ad549382-1e7b-4465-9fe1-fbfe1d1d4110/Screenshot+2023-07-31+094152.png)

If you do not get a bulk token and receive an error, try the following:

-   Log into Azure AD at the brand new **entra.microsoft.com** portal.
    
-   Select **Enterprise applications -> All applications -> Windows Configuration Designer (WCD)**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/71efb2fa-251f-49eb-8abc-68df8002ae09/Screenshot+2023-07-31+at+10.17.14+PM.png)

On the application page select **Permissions** and then click **Grant admin consent for <TENANT NAME>**.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/a375c9d1-77b7-407e-bd3d-9a9e82520058/Screenshot+2023-07-31+at+10.18.39+PM.png)

### SCRIPT VERSION

If you can’t seem to get the the provisioning package created with the Configuration Designer app, you can follow this alternative method. The man himself, [Michael Niehaus](https://www.linkedin.com/in/mniehaus/), wrote a [great piece](https://oofhours.com/2023/02/14/simplify-the-process-of-generating-an-aad-bulk-enrollment-provisioning-package/) on this so I recommend giving it a read. We’re going to use the [AADInternals](https://github.com/Gerenios/AADInternals) PowerShell module with a different version of the Configuration Designer to generate the package.

Download the Windows Assessment and Deployment Kit (ADK) from [here](https://support.microsoft.com/en-us/windows/adk-download-for-windows-10-2a0b7ff2-79b7-b989-f727-43ae506e36ad) and run the installer.

During the installation, at the **Select the features you want to install** screen, you only need to check the “Configuration Designer” option and click “Install”.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7b74cbd2-ac80-4533-9332-0814c5fb9bab/blog3-1.png)

Install and import the **AADInternals** PowerShell module. Like most modules, this is a straight forward process.

```
Install-Module AADInternals -Force
Import-Module AADInternals
```

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/bfd0263c-b8f3-4429-bc1c-95bcdcac6d5e/Screenshot+2023-07-31+093415.png)

_\*We’re going to generate the package with a full PowerShell script later, which will also install and import for us. I personally like to go through the steps piece by piece to get a better understanding of what’s going on._

Here is a breakdown of how the process works with the module:

FIrst, authenticate to Azure with user credentials. The script prompts for username and password.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/80a1b47b-578d-4d3f-a056-b3a7fcef966f/Screenshot+2023-07-31+094513.png)

After authenticating, generate a BPRT and set an expiration date.

Next, we’ll construct the XML used to create the provisioning package and insert our BPRT. This is basically the inverse of what we did using the Configuration Designer, which generated the XML after the package creation.

The nice thing about this is that the XML we’re constructing is identical to the “customizations.xml” generated from the designer.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/044dadd8-5d0a-47aa-b2bb-3fcb3fd47482/Screenshot+2023-07-31+095146.png)

On the left, is the XML we built in the script. On the right is the one generated with the package. So if you’re using this method, you can easily add the settings for the local admin and device name by inserting them into your XML.

_\*I’ll place an XML template in the_ [_migration repo_](https://github.com/groovemaster17/IntuneMigration)_._

With the BPRT generated and placed in the XML, we now simply call the Configuration Designer executable from the ADK, pass our XML file, and choose an output path.

```
ICD.exe /Build-ProvisioningPackage /CustomizationXML:$Filename.xml /PackagePath:$Filename.ppkg
```

Navigate to your output destination and you’ll see a familiar group of files.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/410394ca-eb82-4d60-9d68-561dff2c8413/Screenshot+2023-07-31+094823.png)

The full PowerShell script for generating the package with AADInternals can be downloaded [here](https://oofhours.files.wordpress.com/2023/02/generate-aad-ppkg.zip).

Next time…
----------

Now that we have our provisioning package created, we’ll be able to add our PC to Tenant B. However, when it enrolls to Intune it won’t have a primary user attribute. We’ll need a task to fix that…
