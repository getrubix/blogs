---
author: steve@getrubix.com
date: Mon, 13 Apr 2020 03:31:06 +0000
description: '"Regardless of how you feel about co-management (98% of the time I despise
  it), we have to deal with it. One of the more common situations I run into is when
  an organization is using both co-management in addition to pure, Intune managed
  PCs. There is"'
slug: dynamic-groups-for-co-managed-devices
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/dynamic-groups-for-co-managed-devices_thumbnail.jpg
title: Dynamic groups for co-managed devices
---

Regardless of how you feel about co-management (98% of the time I despise it), we have to deal with it. One of the more common situations I run into is when an organization is using both co-management in addition to pure, Intune managed PCs. There is no clear way to catch all co-managed devices in their own dynamic group. This is a major issue if you're looking to apply separate profiles to newly deployed PCs vs your current fleet.

I've put together a (somewhat) simple solution for dealing with this in the form of a PowerShell script. Using the Microsoft Graph, we can search Azure for all devices enrolled via co-management, create a brand new group, and then use the search results for the new group's members.

Here are a few things to note before we get started:

-   If you're not aware, co-management is the term for using both SCCM and Intune to manage a PC.
    
-   This script uses graph calls that require permissions to read devices, read and write groups, and read directory objects. Make sure to login as an Intune or global administrator.
    
-   This is written entirely out of desperation; if you or someone you know suggest any ways to improve this thing, send it through.
    

Without further ranting, I give you the script:

\## INSTALL AND IMPORT REQUIRED MODULES
$modules = @(
    "Microsoft.Graph.Intune"
)

foreach($module in $modules){
    if(Get-Module -ListAvailable -Name $module){
        Write-Host "$module is installed"
        Import-Module $module
    } else {
        Write-Host "Installing $module module"
        Install-Module $module -confirm:$false -AllowClobber -Force
        Write-Host "$module is installed"
        Import-Module $module 
    }
}


## CONNECT TO MS GRAPH ##########
Write-Host "Connecting to Intune..." -ForegroundColor Cyan
Try{
    Connect-MSGraph -ForceInteractive
    write-host "Success" -ForegroundColor green
}
Catch{
    Write-Host "Error- try logging in again" -ForegroundColor red
}

## GET-COMANAGED DEVICES #########

$comag = Get-IntuneManagedDevice | Where-Object {$\_.deviceEnrollmentType -eq "windowsCoManagement"} | Select-Object -ExpandProperty azureADDeviceId

 
 
 ## GET GROUP INFO ##############

$displayName = Read-Host "Enter group name"

$description = Read-Host "Enter a description for your group"



## CONSTRUCT GROUP JSON ##########
$json = @"
{
  "description": "$description",
  "displayName": "$displayName",
  "mailEnabled": false,
  "mailNickname": "$displayName",
  "securityEnabled": true  
}
"@ 

## POST NEW GROUP ############

$graphApiVersion = "beta"
$Resource = "groups"
$uri = "https://graph.microsoft.com/$graphApiVersion/$($Resource)"

$group = Invoke-MSGraphRequest -Url $uri -HttpMethod POST -Content $json


$groupID = $group.id

$groupID


$AzureDevices = Invoke-MSGraphRequest -Url "https://graph.microsoft.com/$graphApiVersion/devices" -HttpMethod GET
$ADID = $AzureDevices.value



## ADD COMAG DEVICES TO GROUP ##########

foreach($pc in $comag){
    $pc = $ADID | Where-Object {$\_.deviceId -eq $pc} | Select-Object -ExpandProperty id
    $groupJSON = @"

      

"@

    Invoke-MSGraphRequest -Url "https://graph.microsoft.com/beta/groups/$groupID/members/\`$ref" -HttpMethod POST -Content $groupJSON
    Write-Host "$pc added to $groupID group"
}

Before you run off and make your own co-managed Azure group, there are a few pieces I want to touch on:

If it ain’t broke
-----------------

As you can see, this isn’t all just straight graph calls. For gathering the devices, we’re using the **Microsoft.Graph.Intune** module to make the **Get-IntuneManagedDevice** call with some filtering. While we can construct our own calls to do the same thing, there’s absolutely no need since they already exist in the module. So here is where we get those devices and store the returned objects in a variable.

![2020-04-12 22_53_01-● comagGroupMaker.ps1 - Visual Studio Code.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1586746435877-5S9XKWETF2UKZ3SYP8WQ/2020-04-12+22_53_01-%E2%97%8F+comagGroupMaker.ps1+-+Visual+Studio+Code.png)

Name your own group
-------------------

I didn’t think anyone would get a kick out of me coming up with funny names for your co-management groups, so I was considerate enough to allow you to choose your own name using **read-host**. We then take those values and insert them into the JSON block that will be used to make up the group.

The call to make the group is pretty simple. It is a **POST** call using the JSON containing our custom values to fill in the group details. It’s important to note that after the group is created, we want to return it’s object ID from Azure. This will be crucial later when adding members to the group, so we’re storing that in the **$groupID** variable.

![2020-04-12 22_56_36-● comagGroupMaker.ps1 - Visual Studio Code.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1586746693046-23CRVNOEGXZ3DKQC38AI/2020-04-12+22_56_36-%E2%97%8F+comagGroupMaker.ps1+-+Visual+Studio+Code.png)

Object related trickery
-----------------------

This next part is where things got tripped up. When you add members to a group in Azure, it works like this:

Add **<PC%OBJECT%ID%GUID>** to **<%GROUP%OBJECT%ID%GUID>.**

Azure is not interested in the friendly names. Well that’s just fine because earlier when we get our co-managed devices, we can simply get their object ID along with it, right? Wrong!

Even though Intune and Azure work off the same database, they don’t look at the same attributes of the device. When we get our co-managed devices, we’re getting them from Intune, or the **deviceManagement** node of the graph. It’s the only way we can look at the **deviceEnrollmentType** attribute.

Devices from this node have two IDs: the **Intune Device ID** and the **Azure AD Device ID**. So which ID do we use to add group members? If you guessed ‘neither of these two’ then you are sadly correct. Turns out, the only way to add members to groups is with the **Object ID**. And that only comes from Azure AD. But there is some good news.

Azure AD also provides two IDs: the **Device ID** and the **Object ID**. And the **Device ID** from Azure matches the **Azure AD Device ID** from Intune.

![Starting with the Azure AD Device ID from Intune…](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1586747572698-ZT19AZZKGQFH31V0NO5Y/2020-04-12+23_02_36-CLIENT7+_+Hardware+-+Microsoft+Endpoint+Manager+admin+center.png)

Starting with the Azure AD Device ID from Intune…

![…we can follow it straight to the Object ID we need in Azure AD](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1586747595164-HPY621ZVB17K3Y3VCF4W/2020-04-12+23_03_20-Device+-+Microsoft+Azure.png)

…we can follow it straight to the Object ID we need in Azure AD

Connect the dots (or device IDs)
--------------------------------

So there’s the correlation; we just need a way to match them up. The first step is to make sure when we get the comanaged devices, we’re capturing their **Azure AD Device IDs**. Actually, it’s the only attribute about them we really need. Here’s how we did that, going back a few steps:

![2020-04-12 23_16_24-● comagGroupMaker.ps1 - Visual Studio Code.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1586747803569-06WLX9EHL0TSGSVQ4M5H/2020-04-12+23_16_24-%E2%97%8F+comagGroupMaker.ps1+-+Visual+Studio+Code.png)

Now that we have the IDs we need from Intune, we need to find the matching ID in Azure AD and then grab the device’s **Object ID**. While we’re at it, we may as well just add that ID to the group as a member since that’s the point of this insanity. To accomplish that, we’ll use a **foreach** loop with the following logic:

-   For each device in the **$comag** group, get a device in Azure AD where the **Device ID** equals the **Azure AD Device ID**.
    
-   Grab that device and get it’s **Object ID** attribute (which is just **id**).
    
-   Insert the **Object ID** into the JSON body that will construct the graph call content.
    
-   Invoke the graph call to place this **Object ID** into our group.
    

![2020-04-12 23_23_21-● comagGroupMaker.ps1 - Visual Studio Code.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1586748222127-9V6044B5HSX58IQ2PYJA/2020-04-12+23_23_21-%E2%97%8F+comagGroupMaker.ps1+-+Visual+Studio+Code.png)

And there we go! That loop should add each co-managed PC to our new group. As I said before, this came out of necessity, so feel free to tweak and clean up as you see fit.

Using the same logic, I’ve also created a similar script to aggregate all Hybrid Azure Joined PCs. I can’t imagine why you’d need that…
