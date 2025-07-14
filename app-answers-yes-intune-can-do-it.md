---
author: steve@getrubix.com
date: Sat, 03 Aug 2019 23:23:00 +0000
description: 'If I use Intune to manage Windows 10 devices, can I only push apps
  from the Microsoft Windows Store?“This is definitely one of the more common questions
  my team gets asked on a daily (maybe hourly) basis. There is an extreme misconception
  that Intune can only deploy apps'
slug: app-answers-yes-intune-can-do-it
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/app-answers-yes-intune-can-do-it_thumbnail.jpg
title: App answers yes Intune can do it
---

## QUESTION: *If I use Intune to manage Windows 10 devices, can I only push apps from the Microsoft Windows Store?*
---

This is definitely one of the more common questions my team gets asked on a daily (maybe hourly) basis. There is an extreme misconception that Intune can only deploy apps to Windows 10 if they come from the modern app store that Microsoft offers. It would maybe be a serious thought if there were anything in that store someone wanted to use; maybe someday. For now, we can absolutely use Intune to continue to push our standard, “Win32” applications.

**FYI:** 
>Intune was always capable of deploying applications if they were an MSI based installer. That method is still available today.

## Create the app
---

Microsoft created a tool for wrapping complete app installation directories in an Intune friendly format. The tool should be called the “Intune App Wrapper”. But instead, we have the [Microsoft Win32-Content-Prep-Tool](https://github.com/Microsoft/Microsoft-Win32-Content-Prep-Tool)– yes, it rolls right off the tongue. But regardless of it’s name, it does an awesome job.

For this example, we will wrap Adobe Acrobat DC which consists of 8 files required for installation. Follow these steps to prepare the application for deployment:

1. Download the **Intune Win32 App Packing tool** from the link above.
    
2. Unzip and place the **Intune-Win32-App-Packaging-Tool-master** folder in a directory that’s easily accessible like C:\\
    

![screen-shot-2019-08-03-at-6.30.42-pm.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581034701081-NC7JXDWJI9PCH9LQDN24/screen-shot-2019-08-03-at-6.30.42-pm.png)

3. Open the CMD tool as **Administrator.**
    
4. Start the packaging tool by typing the full path of the **IntuneWinAppUtil.exe** and press return.
    
5. Specify the following values: Source folder (**C:\\Adobe**)
    
6. Setup file (**AcroRead.msi**)
    
7. Output folder (**C:\\Adobe**) **NOTE**: This example places the out file in the same folder as the installation directory, but you can choose a different location
    

![screen-shot-2019-08-03-at-6.32.43-pm.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581034764746-6QLQI94STFDM64UAGRI6/screen-shot-2019-08-03-at-6.32.43-pm.png)

8. After the process completes, you will have an .Intunewin file in your output directory
    

![screen-shot-2019-08-03-at-6.32.54-pm.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581034806256-CI79J10YT9NK7SCBQLK7/screen-shot-2019-08-03-at-6.32.54-pm.png)

## Upload to Intune

Now that we have our **AcroRead.msi** package, we can upload the installer to Intune for deployment.

1. Log into [https://portal.azure.com](https://portal.azure.com/) as Intune Administrator and go to **Intune – > Client Apps -> Add +**
    
2. From the Add type dropdown, select **Windows app (Win32)**
    

![screen-shot-2019-08-03-at-6.37.20-pm.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581034838882-K36SAJLJL7LPEBJD8IOU/screen-shot-2019-08-03-at-6.37.20-pm.png)

3. For App package file, browse to the location of the **AcroRead.Intunewin** file we created in the previous section.
    
4. App information can be filled out as needed.
    
5. At the Program menu, enter the Install commands and Uninstall commands for the application.
    

![screen-shot-2019-08-03-at-6.38.11-pm.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581034896839-K8PO6TAPUFEOSF8FUBWT/screen-shot-2019-08-03-at-6.38.11-pm.png)

7. For requirements, fill out as desired. Note that the **Operating system architecture** and **Minimum operating system** are required fields
    
8. In Detection rules, you can either manually add a rule or use a custom script. When manually adding a rule, you can choose from MSI, File or Folder, or Registry item
    

![screen-shot-2019-08-03-at-6.38.42-pm.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581034994224-2BY0MUVMG3PL0HH6YWLH/screen-shot-2019-08-03-at-6.38.42-pm.png)

9. After saving the application settings, you can now proceed to assign the application to the desired groups
    

The whole process is pretty straight forward. Note that when running the tool in command line, you can point the “setup file” attribute to anything; batch files, PowerShell scripts, etc. With this last hurdle out of the way, there is no reason Intune cannot handle software deployment for your Windows 10 fleet.
