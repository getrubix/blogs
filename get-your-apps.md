---
author: steve@getrubix.com
categories:
- intune
- powershell
date: Mon, 17 May 2021 19:07:33 +0000
description: "Have you ever needed to retrieve an application that you deployed with Intune but it’s been about a billion years since you packaged that thing? Yeah, we’ve all been there. Luckily, modern IT guru and MVP Oliver Kieselbach created a utility that allows us to retrieve it."
slug: get-your-apps
tags:
- intune
- script
- powershell
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/get-your-apps_thumbnail.jpg
title: Get your apps back
---

Have you ever needed to retrieve an application that you deployed with Intune but it’s been about a billion years since you packaged that thing? Yeah, we’ve all been there. Luckily, modern IT guru and MVP [Oliver Kieselbach](https://oliverkieselbach.com/) created a utility that allows us to retrieve the .intunewin package and crack it open.

Before we get started, grab the utility [here](https://github.com/okieselbach/Intune/tree/master/IntuneWinAppUtilDecoder) and this accompanying [PowerShell script](https://github.com/okieselbach/Intune/blob/master/Get-DecryptInfoFromSideCarLogFiles.ps1).

In order to retrieve the source or install file of an application deployed via Intune, you must enroll a device with the applications assigned to it and proceed to run the IntuneWinAppUtilDecoder.exe via PowerShell.

Application installers do not stay cached indefinitely, so you will have to enroll a new device in Autopilot.

Log into the PC that is now enrolled in Intune. Make sure you have local administrator rights on the device.

Place the decoder tools somewhere accessible like “C:\\Decoder”:

![Picture1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1621278187613-EE3NB671YL6EPGXF9222/Picture1.png)

Open an elevated PowerShell window:

![Picture2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1621278210412-ZKJ19SVUVL9JWRB5VK13/Picture2.png)

Type cd **C:\\Decoder** to navigate to the files we copied over.

Run the Get-DecryptInfoFromSideCarLogFiles.ps1 script (I’ve renamed mine to ‘decode.ps1’).

![Picture3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1621278298958-M9RBYRT0GQNNQ5L5YBLW/Picture3.png)

Depending on how many applications are needed, the script may run for some time.  When completed, open File Explorer and navigate to the output path specified in the script.  It should be:

**%LOCALAPPDATA%\\Temp**

There you’ll find files in both the .tmp and .tmp. decoded formats.

![Picture4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1621278339421-92JEN8MUQUIX52RLW2JC/Picture4.png)

Right click on the .decoded file and with an extraction tool, like “7Zip”, open the archive.

![Picture5.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1621278373065-BO436FJUROXLEZ75TG8I/Picture5.png)

You can now see the original install files of the application, and extract them.

![Picture6.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1621278640753-Z14UXKYPMEWF3OJL6BQG/Picture6.png)

If you get stuck, hit me up at [steve@getrubix.com](mailto:steve@getrubix.com) and happy decoding!
