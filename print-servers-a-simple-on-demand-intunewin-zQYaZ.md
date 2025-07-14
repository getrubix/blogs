---
author: GuestUser
date: Wed, 14 Feb 2024 00:16:15 +0000
description: '"Printers, printers, printers… some time ago I blogged about a powershell
  command to invoke pnputil and install a printer and its drivers. While there are
  much better solutions like Universal Print and third-party SaaS offerings for large
  quantities of printer, some folks prefer to not add these additional"'
slug: print-servers-a-simple-on-demand-intunewin-zQYaZ
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/print-servers-a-simple-on-demand-intunewin-zQYaZ_thumbnail.jpg
title: Print servers a simple on-demand intunewin
---

Printers, printers, printers… some time ago I [blogged about a powershell command](https://www.getrubix.com/blog/deploy-individual-printers-with-intune?rq=printer) to invoke pnputil and install a printer and its drivers. While there are much better solutions like Universal Print and third-party SaaS offerings for large quantities of printer, some folks prefer to not add these additional components (licensing or cost, perhaps?).

For those who have a relatively small pool of printers, let’s say 40 or less, we could do another kind of script if they are all managed on a server. And don’t worry – we’re not wrapping multiple driver packages.

Let’s make a PowerShell script with… absolutely nothing in it! Well, maybe put a comment or something just for fun – but don’t bother putting any real commands in it.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/69c62aa7-9423-4ea6-aaea-98aaf0c8c090/blog1.png)

…Why? 

I’ll tell you why – we’re going to put all the commands we need right in the install line of our win32 package!

Wrap that ps1 as an intunewin, start the upload, and then configure the following…

**Install command:** powershell.exe -command ‘Write-Host “Installing printer \[printer-name\] – do not close this window…”;Add-Printer -ConnectionName “\\\\server.fqdn.com\\printername”'

**Uninstall command:** powershell.exe -command ‘Remove-Printer -Name “printer-name”’

**Install behavior:** User

**Device restart behavior:** No specific action

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/9ddef4db-9c96-429e-81b8-36f8f74da089/blog2.png)

So again… why do we have to do this? Well, we could have technically put the command in the PowerShell script we wrapped. But – that would require an executionpolicy override. To do an executionpolicy override, you need to be an administrator. System has administrator rights… but does System have access to the print server? The user likely does NOT have administrator rights (I hope)… but the user should have access to the print server.

Since we can’t have the User override the executionpolicy, we can still allow them to execute basic commands that don’t require elevation. Also, since this method doesn’t really do a good job at hiding the PowerShell window, that is why I stick the write-host in the beginning.

Now that we have the solution, let’s add a proper detection rule:

**Rules format:** Manually configure detection rules

**Rule type:** Registry

**Key path:** Computer\\HKEY\_CURRENT\_USER\\Printers\\Connections\\;;SERVERPATH;PRINTERNAME

**Detection method:** Key Exists

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/6abfdc4a-ce37-486f-907c-435e5eec3964/blog_detect.png)

What’s nice about this method is that I can just upload the same .intunewin over and over again, and simply change the package name, install commands, and detection rule. With this package now available in the company portal, the user can go ahead and install their desired printer:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/3603eb96-1abc-4ec2-b3c7-0af93590d242/blog4.png)

The ps1 will likely take up most of the screen when it runs, but it should be relatively quick. Just make sure you have line of site to the print server (perhaps you could add a custom requirement script to run Test-NetConnection?).

Anyway, hope this helps some folks who just want a simple, cloud native solution.
