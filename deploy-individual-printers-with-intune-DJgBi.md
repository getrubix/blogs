---
author: steve@getrubix.com
date: Thu, 09 Jan 2020 19:57:00 +0000
description: '"Jesse here – hijacking the blog again. On my first entry I discussed
  the general practices with PowerShell scripts and Intune, and I provided an example
  of performing drive mappings. Today I wanted to give a quick solution that can help
  deploy printers – specifically on an Azure"'
slug: deploy-individual-printers-with-intune-DJgBi
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/deploy-individual-printers-with-intune-DJgBi_thumbnail.jpg
title: Deploy Individual Printers with Intune
---

Jesse here – hijacking the blog again. On my first entry I discussed the general practices with PowerShell scripts and Intune, and I provided an example of performing drive mappings. Today I wanted to give a quick solution that can help deploy printers – specifically on an Azure AD-joined machine.

Now this solution is ideal for setting up a few individual printers. If you have a decent number of printers and you’re looking for a way to connect your print server to your Azure AD-joined devices, then you can deploy the [Hybrid Cloud Printer Server](https://docs.microsoft.com/en-us/windows-server/administration/hybrid-cloud-print/hybrid-cloud-print-deploy) (there are quite a few configuration steps and requirements for this to work). Technically this solution would allow you to print from anywhere – if you are present on the company network, and if you have a script that can connect to the print server, you can actually utilize this and enter credentials when prompted (you can actually avoid the need for credentials if Windows Hello for Business is properly stood up…).

The other option of course is to deploy a third-party service, such as PaperCut, that can connect the AD print servers to the web – this would allow you to simply push the third-party application to endpoints and automatically deploy the necessary printer connections. All are viable options depending on the need. Having said that, here’s a simple solution to deploy printers individually to endpoints.

We can grab the necessary printer drivers and combine them with this simple 4-line script:

C:\\Windows\\SysNative\\pnputil.exe /add-driver "$psscriptroot\\Driver Folder\\Subfolder\\driver.inf" /install
Add-PrinterDriver -Name "Exact Driver Name"
Add-PrinterPort -Name "10.\*\*.\*\*.\*\*\*" -PrinterHostAddress 10.\*\*.\*\*.\*\*\*
Add-Printer "Printer Name" -DriverName "Exact Driver Name" -PortName "10.\*\*.\*\*.\*\*\*"

This script will invoke pnputil to add the bundled drivers into the Driver Store. After that, it will configure the printerport and the printer itself – pretty simple. Once you have the script and a folder with the necessary drivers, you will wrap both items into an intunewin package (NOTE: because the script is wrapped into a win32 package, the action is run in a 32-bit context – hence why pnputil needs the SysNative in the path instead of System32. Banged my head on this one for a little bit…)

I won’t go through every step of wrapping and uploading the intunewin package, but here’s the important properties you’ll need to fill in:

**Install Command:** powershell.exe -Executionpolicy Bypass .\\printScript.ps1

**Uninstall Command:** \* (Intune needs something here, even if you don’t have an uninstall command. If you want, you could include a separate PowerShell script that will remove the printer)

**Detection Rule Type:** Registry

**Key Path:** Computer\\HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Control\\Print\\Printers\\Name-of-Printer

**Value Name:** blank

**Detection Method:** key exists

And that’s it. The cool thing about this is you can make it available for download in the Company Portal – this way users can browse and install only the relevant printers they need. Give it a nice icon and description and your users should have a pretty simple experience. Enjoy!
