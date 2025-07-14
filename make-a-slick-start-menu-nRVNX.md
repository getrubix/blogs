---
author: GuestUser
categories:
- intune
- powershell
date: Mon, 24 Aug 2020 19:43:20 +0000
description: >
  When deploying devices, customers usually want to customize some of the default
  settings on the machine. This can include wallpaper, start layout, power settings,
  and more. To configure these items during Autopilot, we rely on Michael Niehaus’s
  Autopilot Branding script and wrap it as an intunewin package.
slug: make-a-slick-start-menu-nRVNX
tags:
- intune
- endpoint manager
- script
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/make-a-slick-start-menu-nRVNX_thumbnail.jpg
title: Make a Slick Start Menu
---

When deploying devices, customers usually want to customize some of the default settings on the machine. This can include wallpaper, start layout, power settings, and more. To configure these items during Autopilot, we rely on Michael Niehaus’s [Autopilot Branding](https://github.com/mtniehaus/AutopilotBranding) script and wrap it as an intunewin package.

One of the steps in the Autopilot Branding is to import a start layout XML to the default user account. This can be done by customizing the layout on an existing machine and exporting the layout via the following command:

```
Export-StartLayout -UseDesktopApplicationID C:\Layout.xml
```

I like to use the Desktop Application IDs when possible, as the apps will fill the tiles automatically once they install – using LinkPaths typically requires the application to be installed first during the Enrollment Status Page (ESP). Once we have the XML, we can then utilize the Autopilot Branding script to import it as a default layout. Keep in mind that this method allows the user to ultimately change the layout – if you would like to enforce a partial or full layout via policy, this can be done in the Restrictions configuration profile instead.

 There are a couple of issues that I’ve run into with certain start layouts… secondary Edge tiles, classic Control Panel menu tiles, and even the Company Portal app itself doesn’t seem to want to pin upon installation. Edge tiles I know can have assets imported to work properly, but the Control Panel tiles never import (regardless of using App IDs or LinkPaths), and the Company Portal simply doesn’t pin automatically.

Take a look at this start layout. I omitted some additional categories and tiles that were on this layout, and the goal here was to replace Software Center with Company Portal.\\

<LayoutModificationTemplate
  xmlns:defaultlayout="http://schemas.microsoft.com/Start/2014/FullDefaultLayout"
  xmlns:start="http://schemas.microsoft.com/Start/2014/StartLayout" Version="1"
  xmlns="http://schemas.microsoft.com/Start/2014/LayoutModification">
  <LayoutOptions StartTileGroupCellWidth="6" />
  <DefaultLayoutOverride>
    <StartLayoutCollection>
      <defaultlayout:StartLayout GroupCellWidth="6">
        <start:Group Name="General">
          <start:DesktopApplicationTile Size="1x1" Column="2" Row="0" DesktopApplicationLinkPath="%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\Control Panel.lnk" />          
          <start:DesktopApplicationTile Size="2x2" Column="4" Row="0" DesktopApplicationLinkPath="%ALLUSERSPROFILE%\\Microsoft\\Windows\\Start Menu\\Programs\\Microsoft Endpoint Manager\\Configuration Manager\\Software Center.lnk" />
          <start:DesktopApplicationTile Size="1x1" Column="3" Row="1" DesktopApplicationLinkPath="%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\File Explorer.lnk" />
          <start:DesktopApplicationTile Size="2x2" Column="0" Row="0" DesktopApplicationLinkPath="%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\computer.lnk" />
          <start:DesktopApplicationTile Size="1x1" Column="2" Row="1" DesktopApplicationLinkPath="%ALLUSERSPROFILE%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\Devices and Printers.lnk" />
          <start:DesktopApplicationTile Size="1x1" Column="3" Row="0" DesktopApplicationLinkPath="%ALLUSERSPROFILE%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\Default Programs.lnk" />
        </start:Group>
      </defaultlayout:StartLayout>
    </StartLayoutCollection>
  </DefaultLayoutOverride>
</LayoutModificationTemplate>

You would think that including the LinkPaths for the “Devices and Printers” and “Default Programs” would carry over, but the tiles end up being empty. If you try right clicking and pinning the Default Apps section from the modern Settings application, you end up with the secondary tile format that requires more steps for importing (otherwise the tile will be blank). You would also think to update the System Center to the following for Company Portal, but it doesn’t work:

```
<start:Tile Size="1x1" Column="3" Row="1" AppUserModelID="Microsoft.CompanyPortal_8wekyb3d8bbwe!App" />
```

 To work around this, I decided to create my own custom lnk files and simply point to them in the Layout.xml. Since we need to create the lnk files first, we can put these steps somewhere in the beginning of the branding script. Alternatively, you can create a separate script, wrap it as a win32 app, and make it a dependency for the branding package. I utilized the second option for testing purposes.

After making some custom icons, I included them with the following script and wrapped it all into an intunewin package:

if(!(Test-Path "C:\\Resources")){
    New-Item -ItemType Directory -Path "C:\\" -Name "Resources"
}

Start-Sleep -Seconds 5
Copy-Item -Path "$psscriptroot\\portal.ico" -Destination "C:\\Resources" -Force
Copy-Item -Path "$psscriptroot\\defapps.ico" -Destination "C:\\Resources" -Force
Copy-Item -Path "$psscriptroot\\devices.ico" -Destination "C:\\Resources" -Force

$shortcut = (New-Object -ComObject WScript.Shell).CreateShortcut("C:\\Users\\Default\\AppData\\Roaming\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\Company Portal.lnk")
$shortcut.TargetPath = "C:\\Windows\\explorer.exe"
$shortcut.Arguments = "shell:AppsFolder\\Microsoft.CompanyPortal\_8wekyb3d8bbwe!App"
$shortcut.IconLocation = "C:\\Resources\\portal.ico, 0"
$shortcut.Save()

$shortcut = (New-Object -ComObject WScript.Shell).CreateShortcut("C:\\Users\\Default\\AppData\\Roaming\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\Default Programs.lnk") 
$shortcut.TargetPath = "$($env:SystemRoot)\\System32\\control.exe" 
$shortcut.Arguments = "/name Microsoft.DefaultPrograms" 
$shortcut.WorkingDirectory = "$($env:SystemRoot)\\System32" 
$shortcut.IconLocation = "C:\\Resources\\defapps.ico,0" 
$shortcut.Save()

$shortcut = (New-Object -ComObject WScript.Shell).CreateShortcut("C:\\Users\\Default\\AppData\\Roaming\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\Devices and Printers.lnk") 
$shortcut.TargetPath = "$($env:SystemRoot)\\System32\\control.exe" 
$shortcut.Arguments = "/name Microsoft.DevicesAndPrinters" 
$shortcut.WorkingDirectory = "$($env:SystemRoot)\\System32" 
$shortcut.IconLocation = "C:\\Resources\\devices.ico,0" 
$shortcut.Save()

I was happy to find that I could call on control.exe with the proper arguments for any Control Panel shortcuts – it was also nice to confirm I could create any shortcuts I need for modern apps. I ended up placing these icons in the System Tools folder of the AppData path – this way the icons are not immediately visible in the full app list, and you avoid having two Company Portal shortcuts listed next to each other. Once you wrap the script and icons as an app, you can set the detection rule to any one of the ico files or the shortcuts (or you can general a log file at the end, if desired).

Make sure to set this new package as a dependency for the Autopilot Branding and to add it into your apps list for the ESP block. Also, make sure you update the Layout.xml in the Autopilot Branding, which should now look like this:

<LayoutModificationTemplate
  xmlns:defaultlayout="http://schemas.microsoft.com/Start/2014/FullDefaultLayout"
  xmlns:start="http://schemas.microsoft.com/Start/2014/StartLayout" Version="1"
  xmlns="http://schemas.microsoft.com/Start/2014/LayoutModification">
  <LayoutOptions StartTileGroupCellWidth="6" />
  <DefaultLayoutOverride>
    <StartLayoutCollection>
      <defaultlayout:StartLayout GroupCellWidth="6">
        <start:Group Name="General">
          <start:DesktopApplicationTile Size="2x2" Column="4" Row="0" DesktopApplicationLinkPath="%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\Company Portal.lnk" />
          <start:DesktopApplicationTile Size="1x1" Column="3" Row="1" DesktopApplicationLinkPath="%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\File Explorer.lnk" />
          <start:DesktopApplicationTile Size="1x1" Column="3" Row="0" DesktopApplicationLinkPath="%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\Default Programs.lnk" />
          <start:DesktopApplicationTile Size="1x1" Column="2" Row="0" DesktopApplicationLinkPath="%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\Control Panel.lnk" />
          <start:DesktopApplicationTile Size="1x1" Column="2" Row="1" DesktopApplicationLinkPath="%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\Devices and Printers.lnk" />
          <start:DesktopApplicationTile Size="2x2" Column="0" Row="0" DesktopApplicationLinkPath="%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\System Tools\\computer.lnk" />
        </start:Group>
      </defaultlayout:StartLayout>
    </StartLayoutCollection>
  </DefaultLayoutOverride>
</LayoutModificationTemplate>

Once complete, you should end up with a proper start layout that contains your custom lnk icons. You can ultimately create any shortcuts you need for tiles using this method – the files just have to be somewhere in that Start Menu\\Programs path ahead of time. And yes, I need to update that Company Portal icon to the latest one, but you could use your own company’s icon if you wanted to customize it further. Let me know if this helps or if there are other general customizations you are curious about, thanks!

![Picture1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1598298102344-25SRVFG1AO3NYSIKZNWD/Picture1.png)
