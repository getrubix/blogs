---
author: steve@getrubix.com
date: Wed, 02 Aug 2023 12:12:02 +0000
description: '"Finally. Now that we covered the provisioning package and how our device
  will migrate to Tenant B with no user affinity, let’s talk about the task that sets
  the primary user for Intune.After migration, when you navigate to the PC in Intune,
  the “Primary user” attribute should"'
slug: tenant-to-tenant-intune-device-migration-part-5-the-primary-user
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/tenant-to-tenant-intune-device-migration-part-5-the-primary-user_thumbnail.jpg
title: Tenant to Tenant Intune Device Migration Part 5 The Primary User
---

Finally. Now that we covered the provisioning package and how our device will migrate to Tenant B with no user affinity, let’s talk about the task that sets the primary user for Intune.

After migration, when you navigate to the PC in Intune, the “Primary user” attribute should be empty. Take a look:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/00f91986-2d57-4706-ae42-7919f4939760/Screenshot+2023-08-01+at+3.57.38+PM.png)

This is because it was enrolled by the provisioning package, and while it has a principal in Azure AD, it is not a user. We can further verify this in the registry of the newly migrated PC.

Navigate to _HKLM:\\SYSTEM\\CurrentControlSet\\Control\\CloudDomainJoin\\JoinInfo_. Select the GUID and you should see plenty of information about the recent Azure AD join. Notice the **UserEmail** entry:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/898b0229-ba1b-4e2d-9073-6e8fa7e79e17/1.jpg)

At this point, Intune or Azure AD will not be updating the primary user, because it doesn’t exist. So let’s do it ourselves.

### **SET PRIMARY USER (TRIGGER: 10 MIN AFTER LOGON)**

The logic in this task is fairly simple. We need to figure out who is using the PC, and then tell Intune to make that user the “Primary user”. Let’s break down the process and start with some things we will need:

-   App registration for Tenant B with the following permissions:
    
    -   Device.ReadWrite.All
        
    -   DeviceManagementManagedDevices.ReadWrite.All
        
    -   DeviceManagementManagedDevices.PrivilegedOperations.All
        
    -   DeviceManagementServiceConfig.ReadWrite.All
        
    -   User.ReadWrite.All
        
-   Serial number of PC
    
-   Intune object
    
-   Current, logged in user
    
-   User Azure AD object
    

We need the PC serial number to get the Intune device object. You can get the serial number by using the PowerShell cmd…

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/8ec5b58e-9283-492e-aebd-45528298de9f/Screenshot+2023-08-01+at+4.41.48+PM.png)

…or retrieve it from our **MEM\_Settings.xml** file. Either way will work fine.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/521b8fdf-9255-4105-8bda-2942ea6dea86/Screenshot+2023-08-01+at+5.15.47+PM.png)

Just like in the **GroupTag** script, we’ll use that serial number to query the graph for our Intune object ID.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/add005fc-2136-4202-a7e2-9db74b03016d/Screenshot+2023-08-01+at+5.17.19+PM.png)

Now we’ll need the current logged in username. But wait- how do we get that if we just said that the provisioning package was the primary user to the device? Let’s check somewhere else.

Open up the registry again and navigate to _HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Authentication\\LogonUI_. The entry for **LastLoggedOnDisplayName** should match our user display name in Tenant B.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/80544fd4-1f3e-4f4d-aed7-d51528aad443/2.jpg)

Let’s get the value and search the graph for it:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/718be679-90f9-48be-9907-ebc9e2f8f120/Screenshot+2023-08-01+at+5.18.36+PM.png)

That should return the Azure user object, from which we can pull the ID from. Store that in a variable.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/89aaa202-0d5d-4b52-a4ea-3fd0905f6768/Screenshot+2023-08-01+at+5.19.09+PM.png)

This is where things get weird. We need to post the user ID to the Intune object in the graph, but there’s a unique endpoint for that. We can refer to it as the “deviceUsersUri”:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7827a406-09e8-4d5e-8bce-76ecaa42facb/Screenshot+2023-08-01+at+5.19.35+PM.png)

Think of this as the endpoint that contains that device’s users. That will be the endpoint we make our POST call to.

Now we’ll construct the JSON value. This will be the user object endpoint, so we can concatenate the users endpoint with our ID:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/90cb7d36-e55b-471a-b454-6496473db09c/Screenshot+2023-08-01+at+5.20.08+PM.png)

All that’s left is to POST the json value to the “deviceUsersUri” endpoint.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/78176996-7263-4731-ad6a-1e7b2061a79f/Screenshot+2023-08-01+at+5.20.32+PM.png)

After a few minutes, take a look at the device again in Intune.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/19d0f3c4-99e9-49ee-a2e9-9beeb6d639e3/Screenshot+2023-08-01+at+4.30.52+PM.png)

We did it! And it’s not just the Intune object that has the value. When you navigate to **entra.microsoft.com** and look up the device, you’ll see the **Owner** and **User principal name** are linked to the correct ID.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/574b57a4-bcc9-494e-8fce-4b976fbc2302/Screenshot+2023-08-01+at+4.32.31+PM.png)

Next time…
----------

You can get the full **SetPrimaryUser** script and task files from the [repo](https://github.com/stevecapacity/IntuneMigration). Only two more tasks to go…
