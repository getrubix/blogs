---
author: steve@getrubix.com
date: Thu, 23 Jul 2020 03:15:45 +0000
description: '"Deploying applications to Windows 10 devices from Intune has certainly
  come a long way.&nbsp; We went from single file .MSI installers only, to using PowerShell
  scripts to bring down install bits from blob storage to run locally, all the way
  to full application support with .Intunewin"'
slug: intune-and-package-managers-part-1-chocolatey
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/intune-and-package-managers-part-1-chocolatey_thumbnail.jpg
title: Intune and Package Managers - Part 1 Chocolatey
---

Deploying applications to Windows 10 devices from Intune has certainly come a long way.  We went from single file .MSI installers only, to using PowerShell scripts to bring down install bits from blob storage to run locally, all the way to full application support with .Intunewin packages.

Following in the footsteps of modern management, we can use package managers to deploy software so that IT doesn’t need to maintain a repository of applications anymore.  There are two, very friendly package managers for Windows that we’ll be talking about today: [Chocolatey](https://chocolatey.org) and [Winget](https://github.com/microsoft/winget-cli).  Both of these allow us to install, update and remove applications via a simple PowerShell script.

I’m going to deep dive on how to leverage these platforms with Intune to make applications management that much easier, with this write-up focusing on Chocolatey and a future post talking about Winget.  And while you may not want to use this method for all of your applications, it will absolutely take the sting out of the ones that have caused you grief for years (I’m looking at you, Adobe Acrobat!)

Getting started with Chocolatey
-------------------------------

_\*All information on Chocolatey is found directly on_ [_https://chocolatey.org_](https://chocolatey.org/)

Chocolatey is installed via a simple expression delivered via PowerShell:

```
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

Once Chocolatey is installed on the system, applications can be installed easily by finding the package name:

```
choco install PackageName -y
```

_PackageName_ can be substituted with whichever application you choose- those package names can be found [here](https://chocolatey.org/packages).

If you are trying to quickly install an app on your PC, then you can stop here.  But for deployment via Intune, a bit more work is required.  Here is the flow we’ll be following:

1.  Deploy Chocolatey as stand-alone (optional)
    
2.  Create PowerShell install and uninstall scripts
    
3.  Package scripts for deployment (with [Microsoft Win32 Content-prep tool](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool))
    
4.  Upload and configure application for Intune
    
5.  Assign and enjoy
    

Let’s break down what’s involved in each of these pieces.

### 1\. Deploy Chocolatey as stand-alone (optional)

Wait- this part is optional? I know, that seems weird.  Obviously, we need Chocolatey installed before we can use it to install applications.  But there are two ways to go about this.  First, we can deploy Chocolatey to the machine during Autopilot provisioning so we know it’s installed before subsequent apps are deployed.  It’s probably smart to then set Chocolatey as a dependency for said apps.

Alternatively, because it only takes one line, we can include the line to install Chocolatey in the beginning of each install script.  The logic would check for the existence of Chocolatey first, and if not found, install before deploying the intended application package.  I personally prefer this method as it keeps with the theme of packages being lightweight and streamlined.

If you decide to push Chocolatey stand-alone, simply use the Microsoft Win32 Content-prep-tool to wrap a PowerShell script containing the following:

![2020-07-22 22_01_13-Windows PowerShell ISE.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595472181958-JA7BPWAH7LPIJ077IP7V/2020-07-22+22_01_13-Windows+PowerShell+ISE.png)

There are two changes here to the standard Chocolatey one line installer.  First, we’re explicitly calling ‘Invoke-Expression’ as opposed to the ‘iex’ shortcut.  Secondly, we’re piping the command to ‘Out-Null’.  As covered in a previous post , [Fighting the Enrollment Status Page](https://www.getrubix.com/blog/please-wait), we want the installation to finish before the Intune Management Extension looks for the detection rule.

Use the following parameters to configure the Chocolatey .INTUNEWIN file:

![2020-07-22 22_03_27-GLOBAL - Chocolatey _ Properties - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595472482272-TPZMFD3C58ZI7Z7ZTN4S/2020-07-22+22_03_27-GLOBAL+-+Chocolatey+_+Properties+-+Microsoft+Endpoint+Manager+admin+center.png)

![2020-07-22 22_03_44-Detection rule - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595472498108-LJ39SC7B14BK4093M96K/2020-07-22+22_03_44-Detection+rule+-+Microsoft+Endpoint+Manager+admin+center.png)

### 2\. Create PowerShell install and uninstall scripts

Start by creating a directory to place your PowerShell scripts in.  For this example I’ll be deploying Notepad++.  I’ve created two scripts; InstallNotepad.ps1 and UninstallNotepad.ps1, and have placed them both in the install directory.  Be sure to get the package name from [https://chocolatey.org/packages](https://chocolatey.org/packages).

If you’ve decided to package and deploy Chocolatey stand alone, the install script will look like this:

![single.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595472722629-TLWFDMGPQ905JBLFIOVG/single.png)

As you can see, we’re again adjusting the standard “choco install” commands.  Besides adding the **Start-Process -Wait** component, we’re explicitly calling the full path to choco.exe.  Again, while it may not be required, I have found that the more explicit you are with install scripts, the better the results.

If Chocolatey stand alone is not being deployed first, we will add that logic into the beginning of the script:

![2020-07-22 22_17_46-Windows PowerShell ISE.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595472783737-SGTWYAZ7322DN0DXUEFD/2020-07-22+22_17_46-Windows+PowerShell+ISE.png)

There’s no harm in using this as a template for all Chocolatey apps, since if Chocolatey is detected, the install will simply be skipped.  In fact, as a template, I’ve marked the only component that needs to be changed depending on the package being deployed.

My uninstall script contains the following:

![uninstall.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595472968884-XYE63GBNTD6MC3TSVVCR/uninstall.png)

### 3\. Package scripts for deployment

After the scripts are complete, create the .INTUNEWIN file.  If you haven’t already, download the Microsoft Win32 Content-Prep-Tool [here](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool). The command line to wrap will look as follows:

![2020-07-22 22_25_30-Administrator_ Windows PowerShell.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595473175105-L4RX90INAJVUBNZ759QA/2020-07-22+22_25_30-Administrator_+Windows+PowerShell.png)

Before we upload to Intune, obtain all the required information you will need to configure the application; app description, icon, requirements and detection rule.

### 4\. Upload and configure application for Intune

Here are the parameters I’ve set in Intune for my Notepad++ installation:

![2020-07-22 22_28_43-Add App - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595473286237-SYTPUWUFGBJ4K64Y2KFF/2020-07-22+22_28_43-Add+App+-+Microsoft+Endpoint+Manager+admin+center.png)

![2020-07-22 22_29_34-Add App - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595473301120-F0ZCNWS7MVT010XZ5PNM/2020-07-22+22_29_34-Add+App+-+Microsoft+Endpoint+Manager+admin+center.png)

![2020-07-22 22_30_22-Detection rule - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595473320599-94PI8BT8XLBMNHMAU06O/2020-07-22+22_30_22-Detection+rule+-+Microsoft+Endpoint+Manager+admin+center.png)

### 5\. Assign and enjoy

Simple enough, right?  I’ve made Notepad++ available in my Intune Company Portal as an available application.  All I have to do is click ‘Install’ and the app will deploy silently as with any Intune deployment. 

![2020-07-22 22_34_11-FHMC-0719 - VMware Workstation.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1595473388306-0IMZFJXOJHETTUNXBE2H/2020-07-22+22_34_11-FHMC-0719+-+VMware+Workstation.png)

Next up, we’ll be exploring Microsoft’s own package manager.
