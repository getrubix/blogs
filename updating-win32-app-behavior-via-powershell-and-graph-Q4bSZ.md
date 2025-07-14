---
author: GuestUser
date: Thu, 24 Jun 2021 13:00:53 +0000
description: '"Whenever you upload an intunewin package to Endpoint Manager, you can
  generally go back and modify various settings such as the install command, detection
  rule, and most other properties. However, the one property that seems to become
  locked from any future changes is the Install Behavior.Keep in mind"'
slug: updating-win32-app-behavior-via-powershell-and-graph-Q4bSZ
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/updating-win32-app-behavior-via-powershell-and-graph-Q4bSZ_thumbnail.jpg
title: Updating Win32 app behavior via PowerShell and Graph
---

Whenever you upload an intunewin package to Endpoint Manager, you can generally go back and modify various settings such as the install command, detection rule, and most other properties. However, the one property that seems to become locked from any future changes is the Install Behavior.

Keep in mind - the Install behavior is normally determined by the EXE or MSI you are wrapping, and the property is often locked to System or User based on how the software was packaged. The goal isn’t to override this behavior necessarily, and the apps are usually locked one way or the other to install properly.

Having said that; when you wrap an application with a script, if you point to the script instead of the original installer file, MEM will let you chose whether the install behavior is System or User. See the screenshots below:

![Screen Shot 2021-06-21 at 1.45.06 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1624539465543-X8FQ6RIM9IDN17DZMTDM/Screen+Shot+2021-06-21+at+1.45.06+PM.png)

Now let’s say I uploaded this and decided to test the installation behavior. I come to realize that due to some errors in the installation logs, the process needs to run as the User instead of System. If I go back to the same intunewin package, I can not modify the property:

![Screen Shot 2021-06-21 at 1.48.41 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1624539491572-MKPPS6YFMGKN882SVGZA/Screen+Shot+2021-06-21+at+1.48.41+PM.png)

Shoot. I proceeded to try and upload a new intunewin to the existing package, but it still had the property locked. This was something that I needed to change for seven different version of the application, as they had sightly altered settings for different user-based groups. I could have re-wrapped the packages, but I wanted to see if there was any way to just update the package via the graph.

As it turns out, there is a graph call to update this property - I decided to run the command using PowerShell. While I am not including the entire script to authenticate to the graph (there are plenty of examples out there), here is the body and the graph call to post the updated information:

$allApps = Invoke-RestMethod -Uri "https://graph.microsoft.com/beta/deviceAppManagement/mobileApps" -Method Get

$body = @"
{
 "@odata.type": "#microsoft.graph.win32LobApp",
 "installExperience": {
 "runAsAccount": "user",
 "deviceRestartBehavior": "basedOnReturnCode"
 }
}

"@
foreach($app in $allApps){
 if($($app.displayName).Contains("ZTD-AppName-“)){
 $id = $app.id
 Invoke-RestMethod -Uri "https://graph.microsoft.com/beta/deviceAppManagement/mobileApps/$($id)" -Method Patch -Body $body
 Write-Host "$($app.display) is now set to user context"
 }
}

We can initially query for all Intune applications and assigned it to the $allApps variable. The $body variable is the proper json format needed to patch the application package, and we proceed with the correct graph call only for the applications that match the specific display name.

This was to specifically alter a package that contained a script as the primary installation mechanism. This change ultimately allowed me to install my application with success, but I can not say what the effects would be if applications that did not have an accompanying script (if the installer originally let you choose System or User, then I would imagine this change could work here as well). Anyway, hope this helps for some extra app management testing. Enjoy!
