---
author: steve@getrubix.com
date: Wed, 23 Dec 2020 20:51:05 +0000
description: '"It goes without saying that building a Microsoft Endpoint Manager environment
  takes time.&nbsp;&nbsp;Most of that time comes from testing device enrollments to
  ensure all of the profiles and applications we''ve configured are deploying correctly.&nbsp;Intunewin
  applications can be especially tricky if you''re still in the process of figuring
  out"'
slug: intune-app-testing-and-time-travel
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/intune-app-testing-and-time-travel_thumbnail.jpg
title: Intune App Testing and Time Travel
---

It goes without saying that building a Microsoft Endpoint Manager environment takes time.  Most of that time comes from testing device enrollments to ensure all of the profiles and applications we've configured are deploying correctly. Intunewin applications can be especially tricky if you're still in the process of figuring out things like the install line or detection rule.  So to save some time, I've developed a process to test an application deployment on a virtual machine, with the result being as close as possible to an Intune enrollment. Minus the time between device enrollments.

What you’ll need
----------------

-   Windows 10 virtual machine (preferably the same build you’ll be deploying to with Intune)
    
-   PSTools
    
-   Checkpoint or snapshot of the virtual machine
    

Getting the VM ready
--------------------

The state of the virtual machine is the most crucial piece to the time savings.  In this case, I have a Windows 10 virtual machine running the appropriate build to match what I need to test the app deployment with.  Here, it is 20H2 (19042.630).

Next, I've downloaded PSTools, which can be found [here](https://docs.microsoft.com/en-us/sysinternals/downloads/pstools).

I've extracted it to my C drive so it's accessible.  There is also a "Test" folder that will serve as the application install directory.  Let's try installing Adobe Acrobat Pro.  We'll copy all of the install files we need into “C:\\Test”.

![1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1608755756190-848TJEHEFNKU5MIZS2S3/1.png)

Make any other changes or customizations you want to the VM such as GPO, UAC settings, shortcuts, etc.  For example, I turn off the UAC settings and configure PowerShell and CMD to always launch as Administrator.  Remember, we're going for time savings.

With everything all set the way you prefer, go ahead and launch an elevated cmd prompt and type:

```
C:\PSTools\PsExec.exe -i -s cmd
```

![2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1608755932536-MPS30MFQUNY4JQQSWUYM/2.png)

PSTools will launch another CMD session, this time as nt authority\\system.  This is the same context in which Intune will install a system application, so this will give us the most accurate result.

![3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1608755964854-9OMB5YNZA49DVEVRYG9C/3.png)

Go ahead and navigate to your install directory by typing:

```
cd C:\Test
```

Snapshot
--------

I'm going to create a Hyper-V checkpoint to snapshot the current state of the VM,  because this is the absolute perfect spot to do so.  My install files are ready, I have CMD running as system to mirror Intune, and I'm sitting in the install directory.  Now if something goes wrong, I can return to this exact moment in time.  I suppose it's like _Back to the Future_, just not as entertaining or cool.

![4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1608756048443-RX2M6VZTQ1ARRMN40DLS/4.png)

Installing the app
------------------

Now that I've frozen that moment in time, it's time to install the app.  Keep in mind, that whatever you type at this point should be the same as what you intend to configure for the install command in Intune.

![5.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1608756085711-89QP8K919GXL2YNNWKAL/5.png)

![6.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1608756096637-HOZWD3CMOGCMO9GDIJOK/6.png)

Hopefully, the app will install.  If it does, go ahead and package using the Win32 Content Prep Tool.  Here's my [last writeup](https://www.getrubix.com/blog/app-answers-yes-intune-can-do-it) on it.

But if for some reason things don't go as planned, you can now snap back to where you need to be.
