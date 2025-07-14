---
author: steve@getrubix.com
date: Mon, 13 Sep 2021 16:07:34 +0000
description: '"This week, I spent a decent amount of time walking folks through my
  Windows 365 lab setup and how it’s managed with Microsoft Endpoint Manager (or Intune).
  The response has been very positive and many organizations are eager to get started
  with a POC or implementation.There is"'
slug: windows-365-endpoint-manager-a-deeper-look
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/windows-365-endpoint-manager-a-deeper-look_thumbnail.jpg
title: Almost Autopilot Windows 365 and the Enrollment Status Page
---

This week, I spent a decent amount of time walking folks through my Windows 365 lab setup and how it’s managed with Microsoft Endpoint Manager (or Intune). The response has been very positive and many organizations are eager to get started with a POC or implementation.

There is one question that I heard in almost every conversation; “can I use Autopilot to deploy a Windows 365 PC?”

My knee-jerk reaction was “of course not- why would you want to?” Not to mention the fact that Cloud PCs don’t start in an out-of-box state. But then I thought a bit about what was being asked. It wasn’t actually Autopilot that folks wanted, but more the end user experience. They wanted to be able to:

-   Block a device from being used until all of the required applications and policy settings were applied
    
-   Display the status of the provisioning process to the end user while things were installing
    
-   Have everything ready for the end user as soon as they reach the desktop
    

So, this wasn’t really Autopilot they wanted. It was the Enrollment Status Page (ESP). All we have to do is get it working with Windows 365 Cloud PCs. That should be easy right?

Cross the streams
-----------------

Some have told me that I treat the ESP as an arch nemesis. That seems a bit extreme. My thoughts are the ESP is the one part of Endpoint Manager with the narrowest margin for error. If you want a great user experience, then the ESP must be perfect.

If you haven’t set the ESP up in your environment, you should absolutely do that this minute. It’s fairly simple. Log into [https://endpoint.microsoft.com](https://endpoint.microsoft.com) and navigate to **Devices -> Enroll devices -> Windows enrollment -> Enrollment Status Page**.

![2021-09-12 18_22_33-Enroll devices - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1631546983446-8DG2FNDASLOWDCZARS5D/2021-09-12+18_22_33-Enroll+devices+-+Microsoft+Endpoint+Manager+admin+center.png)

There is always one, default ESP configuration in my tenant. Adding more than one used to cause significant issues in Autopilot deployment including extreme delays in provisioning or just straight up time outs. To read more about it, check out the [PSA wrote a while back](https://www.getrubix.com/blog/multiple-enrollment-status-pages-a-psa). Just like when Egon told the other Ghostbusters to “never cross the streams” with their proton packs, I have made sure my team and I only implement one ESP config.

However, it seems that needs to change in order to push an ESP to the Windows 365 PC. So to make this work, I would have to cross the streams.

I went ahead and created a new ESP configuration and assigned it to my Cloud PC dynamic group that captures all my Windows 365 machines.

![2021-09-12 18_25_30-Enrollment Status Page - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1631547221208-FM8DH768BE6KUG8W5QIM/2021-09-12+18_25_30-Enrollment+Status+Page+-+Microsoft+Endpoint+Manager+admin+center.png)

The main difference is that I have configured the setting **Only show page to devices provisioned by out-of-box experience (OOBE)** to **No**. I have this set to **Yes** in the default ESP so that only Autopilot devices see it. This is useful for co-managed devices or other PCs that don’t need to go through the OOBE workflow.

But Windows 365 PCs technically aren’t provisioned by OOBE and I still want them to receive an ESP. So setting that policy to **No** should do the trick.

![2021-09-13 10_54_10-Cloud PC ESP - Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1631547396880-9E7HUPQZYAO09PPK0YIO/2021-09-13+10_54_10-Cloud+PC+ESP+-+Microsoft+Endpoint+Manager+admin+center.png)

Don’t skip the user
-------------------

Another constant in my Endpoint Manager setup is the policy to skip the User Account Setup status portion of the ESP. For more information on why that’s the case, see my [full post here](https://www.getrubix.com/blog/please-wait).

For our Cloud PCs, however, we would need to show the User Account Setup status. This is mainly due to the fact that Windows 365 goes through the device setup process during provisioning, so it stands to reason that there would be nothing to display during Device Setup (I’ll dig deeper on this and keep everyone updated).

My policy to disable the User Status setup was set to all devices. I changed the assignment to only Autopilot PCs. That should leave my Windows 365 fleet excluded from the policy, right? Wrong.

Thankfully, I received a tip from Steve DeQuincey at Microsoft. What I needed to do was not just exclude the Cloud PCs from the policy to block the User Setup status, but also configure a new policy to _un-block_ it. Here is the custom configuration profile I made:

**OMA-URI:** ./Vendor/MSFT/DMClient/Provider/MS DM Server/FirstSyncStatus/SkipUserStatusPage

**Data type:** Boolean

**Value:** False

I would then assign this to the Windows 365 Cloud PC dynamic device group.

Assign things to the user
-------------------------

Lastly, there should be some applications and policy assigned to the user groups logging into the Cloud PC. Not only is this required to track in the ESP, but it makes sense when you consider the nature of Windows 365.

There should be one set of applications and policy targeted to all Cloud PCs. Then, you would want to deploy specific programs and settings based on the user assigned to the machine. This will allow you to streamline the Windows 365 provisioning process while providing different users with specific configurations based on their persona.

So, I made sure the user group had some applications and profiles assigned.

Go time
-------

My configuration in Endpoint Manager was complete. I setup a new ESP, configured the User Setup policy for the Cloud PCs, and assigned applications and profiles to the user group who would be logging in.

I went ahead and provisioned a new Windows 365 PC for a user in the group. As that user, I logged into [https://windows365.microsoft.com,](https://windows365.microsoft.com,) launched the Cloud PC, and BOOM:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1631548208726-ONP66JUHG2DPYY1BO5ZO/2021-09-13+10_29_46-Remote+Desktop+Web+Client.png)

Just like an Autopilot device, I was met with the ESP to display my provisioning progress. I’m still curious has to why **Device preparation** and **Device setup** are waiting for the “previous step”… I’ll dig through those pieces later. But it worked exactly as I wanted.

And now, once the user got to the desktop, everything was installed and ready to go:

![2021-09-13 10_38_02-Remote Desktop Web Client.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1631548341156-TPUCNEYM4J9066HQEDVT/2021-09-13+10_38_02-Remote+Desktop+Web+Client.png)

Autopilot or not?
-----------------

So is this Autopilot for Windows 365 PCs? Not really. But by configuring the ESP to work here, we’ve achieved the following:

-   Consistent sign in experience regardless of PC type
    
-   Transparent setup status during provisioning
    
-   Fully deployed workspace ready to go for the end user with all apps and policy in place
    

So while it’s not technically Autopilot, it is a streamlined deployment process and a great end-user experience. And hey, isn’t that what Autopilot is all about?
