---
author: steve@getrubix.com
categories:
- intune
- powershell
- automation
- azure
date: Thu, 03 Aug 2023 12:00:36 +0000
description: >
  Here we go, folks – we’re at the last two tasks in the migration process. After this, we will have all the pieces needed to test this out. Who’s excited?
slug: tenant-to-tenant-intune-device-migration-part-6-the-last-tasks
tags:
- azure
- aad
- script
- powershell
- intune
- automation
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/tenant-to-tenant-intune-device-migration-part-6-the-last-tasks_thumbnail.jpg
title: Tenant to Tenant Intune Device Migration Part 6 The Last Tasks
---

Here we go, folks- we’re at the last two tasks in the migration process. After this, we will have all the pieces needed to test this out. Who’s excited?

### **AUTOPILOT REGISTRATION (TRIGGER: 30 MIN AFTER LOGON)**

By this point, our device has migrated into Tenant B and should be managed by Intune. It should also be completely removed from Tenant A by now. We want to register it with Autopilot in Tenant B so we have the ability to re-deploy as needed.

While we’re using the same task and script process for this, the script itself will be a bit different.

We cannot add a device to Autopilot with the usual “Invoke-RestMethod” call to the graph API, so we need to use a module. The first part of our script installs and imports both the **Microsoft.Graph.Intune** and **WindowsAutopilotIntune** PowerShell modules.

The modules also vary in the way they authenticate. We can use the same app registration for Tenant B as previous scripts, but we also need the tenant ID. Here is what the app info looks like and the line we use to authenticate to graph with it:

```
$clientId = "<CLIENT ID>"
$clientSecret = "<CLIENT SECRET>"
$clientSecureSecret = ConvertTo-SecureString -String $clientSecret -AsPlanText -Force
$clientSecretCredential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $clientId,$clientSecureSecret
$tenantId = "<TENANT B ID>"

Connect-MgGraph -TenantId $tenantId -ClientSecretCredential $clientSecretCredential
```

Now that we’re authenticated, we need three things to import the device to Autopilot- the hardware ID, serial number, and group tag.

_\*While we added the group tag manually to the Azure AD object previously, we’re going to still attach it here so it stays associated with the Autpilot entry._

The hardware ID and serial number are retrieved with wmi-object calls and the tag we can still get from our **MEM\_Settings.xml** file.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e6b7c77e-8ace-4173-b106-71da2ef262e4/Screenshot+2023-08-02+at+2.11.17+PM.png)

To import the device, just pass those three values to the cmd in the Autopilot module:

```
Add-AutopilotImportedDevice -serialNumber $ser -hardwareIdentifier $hwid -groupTag $tag
```

### **MIGRATE BITLOCKER KEY (TRIGGER: 30 min after logon)**

The final step is to take the current BitLocker key of the PC and escrow that to Azure AD in Tenant B.

Start by retrieving the BitLocker volume for the “C:\\” drive and store it in a variable.

```
$BLV = Get-BitLockerVolume -MountPoint "C:"
```

Now we’ll use the “BackToAAD” cmdlet start the escrow to Azure AD.

```
BackupToAAD-BitLockerKeyProtector -MountPoint "C:" -KeyProtectorId $BLV.KeyProtector[1].KeyProtectorId
```

Give it a few minutes and then check on the device in Intune. Navigate to **Recovery keys -> Show Recovery Key** to verify the ID and key are now present.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1b2dfa71-3a4d-477d-a5e3-7bee31128881/1.jpg)

Next time…
----------

We made it! At this point we have all the tasks and scripts needed to migrate. Feel free to grab everything from the [Intune Migration repo](https://github.com/stevecapacity/IntuneMigration). Next, we’ll build the application package and get ready to test!
