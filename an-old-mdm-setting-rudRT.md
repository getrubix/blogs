---
author: GuestUser
categories:
- intune
- security
- powershell
date: Thu, 02 May 2024 20:04:51 +0000
description: >
  Every now and then, believe it or not, we still come across tenants that do NOT
  have the MDM Authority set to Intune. You may encounter this if you can’t edit
  the Enrollment Status Page (ESP) or Enrollment restrictions profiles - this of
  course needs to be updated.
slug: an-old-mdm-setting-rudRT
tags:
- intune
- security
- script
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/an-old-mdm-setting-rudRT_thumbnail.jpg
title: An Old MDM Setting
---

Every now and then, believe it or not, we still come across tenants that do NOT have the MDM Authority set to Intune. You may encounter this if you can’t edit the Enrollment Status Page (ESP) or Enrollment restrictions profiles - this of course needs to be updated in order to fully manage your devices.

You may notice however that [previously-documented instructions](https://learn.microsoft.com/en-us/mem/intune/fundamentals/mdm-authority-set) are no longer valid, as there is no orange banner or menu you can manually navigate to change the setting. Even though updated tenants should have it set automatically, there are random cases where it never got configured.

Rudy Ooms previously posted a [great blog](https://call4cloud.nl/2021/01/intune-battle-of-the-mdm-authority/) detailing this issue and a scripted solution. I went ahead and updated his script to a module-less version with an app registration. You’ll of course need to throw in your tenant id, application id, and secret value, and the app registration will need granted consent for the permissions listed in the comments.

###############  App registration / token  ##################

\[Net.ServicePointManager\]::SecurityProtocol = \[Net.SecurityProtocolType\]::Tls12
Add-Type -AssemblyName System.Web

#APPLICATION-BASED PERMISSIONS NEEDED (least to most privileged):
#DeviceManagementServiceConfig.ReadWrite.All, DeviceManagementConfiguration.ReadWrite.All

#App registration
$tenantId = ""
$clientId = ""
$clientSecret = ""
$clientSecret = \[System.Web.HttpUtility\]::UrlEncode($clientSecret)

#Header and body request variables
$headers = New-Object "System.Collections.Generic.Dictionary\[\[String\],\[String\]\]"
$headers.Add("Content-Type", "application/x-www-form-urlencoded")
$body = "grant\_type=client\_credentials&scope=https://graph.microsoft.com/.default"
$body += -join("&client\_id=" , $clientId, "&client\_secret=", $clientSecret)
$response = Invoke-RestMethod "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token" -Method 'POST' -Headers $header -Body $body
$token = -join("Bearer ", $response.access\_token)
#Reinstantiate headers
$headers = New-Object "System.Collections.Generic.Dictionary\[\[String\],\[String\]\]"
$headers.Add("Authorization", $token)
$headers.Add("Content-Type", "application/json")

##############################################################

# Update the MDM Authority to Intune

try 
{
    Invoke-RestMethod -Uri "https://graph.microsoft.com/v1.0/organization/$tenantId/setMobileDeviceManagementAuthority" -Method 'POST' -Headers $headers
    Write-Host "MDM Authority updated to Intune."
}
catch 
{
    $message = $\_.Exception.Message
    Write-Host "Failed to set MDM Authority: $message"
}
