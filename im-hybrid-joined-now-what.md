---
author: steve@getrubix.com
date: Wed, 05 Feb 2020 20:13:00 +0000
description: '"You’ve read the documents, setup the Intune connector, created the
  OU and delegated the privileges (and if you haven’t,&nbsp;read my hybrid join post).
  The dynamic group that is set to catch all of your ‘hybrid’ tagged PCs seems to
  be working. You power on the PC, see that"'
slug: im-hybrid-joined-now-what
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/im-hybrid-joined-now-what_thumbnail.jpg
title: Im Hybrid Joined Now What
---

You’ve read the documents, setup the Intune connector, created the OU and delegated the privileges (and if you haven’t, [read my hybrid join post](https://z0touch.home.blog/2020/01/25/autopilot-for-hybrid-join-a-somewhat-visual-guide/)). The dynamic group that is set to catch all of your ‘hybrid’ tagged PCs seems to be working. You power on the PC, see that custom OOBE screen, and proceed with the enrollment. Just like magic, it looks like you’re on the desktop- congratulations! You have successfully used Autopilot to Hybrid Azure AD join a PC… now what?

Did it work?
------------

One of the funniest things I get asked about a hybrid joined PC is “how do I know if it worked?” I completely get why it would be confusing. At first glance, an Autopilot machine that has been hybrid joined looks exactly like one that was Azure AD joined. But there’s a few things we can check to make sure the magic actually occurred.

### Start with the device

Our first step would be to check the PC itself. Open the command prompt as the current user (no need for administrator). type _dsregcmd /status_ and press **Enter**. Out of all the output that gets created, there are two key areas. First is the “device state”:

![2020-02-04-09_02_51-batcave-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581106263582-LT025XDAF8HEEORVDU2U/2020-02-04-09_02_51-batcave-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png)

Nothing left to interpretation there; the device shows it is both Azure AD joined and Domain joined. The local domain name is verified as well. Equally as important is the value _AzureAdPrt_. This stand for the Azure Primary Refresh Token, which is the main authentication factor for SSO in Azure AD. Make sure the value is YES:

![2020-02-04-09_03_00-batcave-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581106293921-YY35NY1NBFGFN5MFSQK9/2020-02-04-09_03_00-batcave-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png)

If the value is NO, then we need to look if the user is licensed for EMS (which includes Azure Active Directory Premium).

Another place to check on the device itself would be **Settings -> System -> About**. The actual computer name should be updated with your specified prefix (from the Intune domain join profile) along with a random string. Take a look:

![2020-02-04-09_03_47-batcave-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581106308892-4Q25MV46Q2MR9AADL7NV/2020-02-04-09_03_47-batcave-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png)

I’ve also highlighted my Windows edition, which is Windows 10 Enterprise. The machine started life as Windows 10 Pro, but found my user assigned Windows Enterprise license attached to the ID that I enrolled with. This behavior is also dependent on the Primary Refresh Token, so if that fails, so does this.

### Up in the cloud

Our next stop is going to be in Azure Active Directory. Go ahead and log into [https://portal.azure.com](https://portal.azure.com/) with admin credentials. Navigate to **Azure Active Directory -> Devices -> All devices** and search for the device name (that we’ve identified in the previous step). Wait a minute… why are there two?

![2020-02-05-18_34_41-device-azure-active-directory-admin-center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581106328666-C4ZQFJ7I7LKNI35JEP7Q/2020-02-05-18_34_41-device-azure-active-directory-admin-center.png)

Don’t be alarmed, this is normal behavior. What happened is the Azure AD joined device object was created when we registered in Autopilot and enrolled to Intune via OOBE. But then the device domain joins, and Azure AD Connect will sync it back up to Azure AD as **Hybrid Azure AD Joined** (usually within 30 min).

While the two objects will remain in Azure, only the Hybrid joined device is used by Intune for profile and applications assignment. In fact, Intune should only be showing you one device from here on out.

### Back at the ranch

Our last validation point is the local Active Directory domain- I mean, can you think of a better place to see if the domain join actually took place?

Log onto a domain controller with admin credentials and launch **Active Directory Users and Computers**. Click on the specified OU for your Hybrid joined devices.

![2020-02-05-18_48_07-z0tdc03-z0tdc03.zerotouch.local-remote-desktop-connection-manager-v2.7.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581106359797-WHR2DJQTQZEQVW1L1OQW/2020-02-05-18_48_07-z0tdc03-z0tdc03.zerotouch.local-remote-desktop-connection-manager-v2.7.png)

You should see the new PC sitting there with the correct name prefix- in my case, _ZTD_. From there, you’re free to move the device to different OUs in order to get appropriate group policy.

Hopefully, your device has passed all of these validation points. Let me know how it goes and if you have further questions- happy testing!
