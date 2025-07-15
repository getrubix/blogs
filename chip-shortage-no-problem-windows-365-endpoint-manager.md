---
author: steve@getrubix.com
date: Thu, 02 Sep 2021 17:42:13 +0000
description: '"Despite the challenges around moving to the cloud in addition to functioning
  amid a global pandemic, many organizations continue to hire at an exponential rate.
  You’ve got Autopilot and Intune setup to onboard Windows devices for all the new
  employees starting. There’s just one problem: a"'
slug: chip-shortage-no-problem-windows-365-endpoint-manager
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/chip-shortage-no-problem-windows-365-endpoint-manager_thumbnail.jpg
title: Chip shortage No problem Windows 365 Endpoint Manager
---

Despite the challenges around moving to the cloud in addition to functioning amid a global pandemic, many organizations continue to hire at an exponential rate. You’ve got Autopilot and Intune setup to onboard Windows devices for all the new employees starting. There’s just one problem: a major hardware shortage. As if on cue, Microsoft presented a solution for companies of all sizes to get their employees up and running; chip shortage or not.

## Enter Windows 365
---

Here’s the official description per Microsoft:

> Windows 365 combines the power and security of the cloud with the versatility and simplicity of the PC. From contractors and interns to software developers and industrial designers, Windows 365 enables a variety of new scenarios for the new world of work.

In short, it is a cloud PC that can be deployed with minimal effort. For everything you need to know about it, I recommend checking out this [great write-up by getnerdio.com.](https://getnerdio.com/academy-enterprise/microsoft-windows-365-vs-azure-virtual-desktop-avd-comparing-two-daas-products/)

At this point, you may be thinking to yourself “hey Steve, why are you writing about virtual PCs? I thought endpoint management is your area?” While that’s valid, the best part of Windows 365 is the inherent management capability. To start, Windows 365 is administered right in the Endpoint Manager admin center.

![Devices view in Endpoint Manager](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1630597456911-4XDKL1YZD2RYZ99L2FAE/2021-09-02+10_00_29-Devices+-+Microsoft+Endpoint+Manager+admin+center.png)

I’m not going to spend time on all the official ‘getting-started’ shtick; head over to the [Microsoft documents](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/get-started-with-windows-365-enterprise/ba-p/2530504) for all that goodness.

Let’s jump into how to manage these things.

## Join 'em
---

The first question is the most obvious; how do we join these devices to our environment? Can they be domain joined, Azure AD joined, or Azure registered? As of this writing, the Windows 365 Enterprise PCs can only be Hybrid Azure AD joined. Support for Azure AD is inbound for the end of 2021 (which probably means first half of 2022). There is another SKU for Windows 365 Business that is only Azure AD joined, but offers far less management capability, so I won’t be discussing it.

The initial setup involves creating an Azure VNET with connectivity to your domain. My “physical” domain resides in Azure, so that was fairly simple. With a traditional on-premises domain, you need to create a site-to-site connection. More on that can be found here: [Tutorial: Site-to-site portal](https://docs.microsoft.com/en-us/azure/vpn-gateway/tutorial-site-to-site-portal).

Once there is an available VNET, you configure the On-premises network connection settings with the network info and domain join information.

![Network configuration](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1630598349308-8J64RNZ4E66LAN2GIEBP/2021-09-02+10_06_29-NTWRK-ZTDSCLOUD-W365+-+Microsoft+Endpoint+Manager+admin+center.png)

Windows 365 PCs are purchased just like any other M365 license and assigned to a user or group the same way. There are various licenses for different hardware specs.

![License store UI](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1630598513041-02YVU6Z6EGTWGCG254YR/2021-09-02+11_16_35-Microsoft+365+admin+center.png)

Welcome to the computer store of the future

When a new machine is provisioned, it is automatically Hybrid Azure AD joined and enrolled in Endpoint Manager. So how exactly can I manage this device? Well, you manage it exactly like any other Windows device.

## Keep them together
---

Before applying any configuration, I recommend creating an Azure dynamic group for Windows 365 PCs. Azure has a specific property you can use to do this. Use a dynamic query like:

```
(device.deviceModel -contains "Cloud PC")
```

Now every time a new machine is provisioned, it will join the group.

![Cloud PC group creation](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1630598788490-F34L2RCMURXYR0H7Y3UY/2021-09-02+10_07_24-Windows+365+Cloud+PCs+-+Microsoft+Endpoint+Manager+admin+center.png)

Dynamic groups are always a smart idea in Endpoint Manager so we can explicitly target different builds with different policy.

Back “on the ground”, I would also suggest creating an OU in Active Directory for the same reasons. As you can see, I got real creative with mine:

![Active Directory OU](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1630598945811-OQQNVF0880XEYFTPLXYJ/2021-09-02+10_12_24-ZTDSDC01.png)

## Let Intune do its thing
---

From here on, we can manage these just like any other PC enrolled in Endpoint Manager.

![Intune-managed device](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1630603274984-SLSMJA9CE1WKF40LOBTI/2021-09-02+10_07_58-CPC-w365-1YW-RV+-+Microsoft+Endpoint+Manager+admin+center.png)

See anything out of the ordinary? That’s right- you don’t. Because it’s exactly the same (except for encryption, but that’s to be expected with any virtual machine).

Just with my other machines, I have device configuration policy applied.

![Device configuration settings](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1630603574763-RTEVWYEFSJDME1BYYG44/2021-09-02+10_08_33-CPC-w365-1YW-RV+-+Microsoft+Endpoint+Manager+admin+center.png)

You can see my domain certificates, device settings, endpoint protection and even my defender for endpoint onboarding policy have all been applied by targeting them to the dynamic group that was created.

The result on the Windows 365 PC is consistent as well.

![Remote Desktop Web Client view](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1630603632093-7129ICZHLXSFXFU6SP4O/2021-09-02+10_11_05-Remote+Desktop+Web+Client.png)

Policy is applied immediately. As shown, Cortana is disabled, the “allow private store only” policy for the app store is applying, and Windows Updates are being managed per my Update for Business rings.

## How about a side order of Group Policy?
---

Remember, this device is Hybrid Azure AD joined, which means it does exist in my on-premises Active Directory. I can apply any group policy object to it in addition to Intune configuration.

![Group Policy in AD](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1630603806747-W8Q26HM1T3J3FT4FK9SV/2021-09-02+10_12_39-ZTDSDC01.png)

In this case, the policy includes some basic domain settings, in addition to a network drive being mapped. The truth is I forgot about this until I opened the file explorer…

![Mapped network drive](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1630603881533-BVE7GF6R1JXOZVEMPCFL/2021-09-02+10_25_01-Remote+Desktop+Web+Client.png)

Boom! There’s my H Drive, mapped automatically to the Windows 365 PC. No additional steps were needed.

## So what’s next?
---

Windows 365 Cloud PCs were officially launched on August 2nd of this year. As you can imagine, we’re barely getting started with exploring all the capabilities. But let’s recap what just happened. After a straight forward Azure network setup, I purchased a license and assigned it to a user, same as an Office 365 license. A dynamic group was created to catch all Cloud PCs. After that, we added that group to some existing policy assignment.

Now I have a provisioned desktop that is user ready with all their M365 services, but it is completely managed to the posture of my current Windows 10 fleet. And, I honestly never dabbled much with VDI in the past.

So if you deploy Endpoint Manager today and have no plans of slowing down your hiring (kudos by the way), then it’s worth checking out Windows 365. Chances are most folks have something to run it on, be it an iPad, home PC or a Chromebook. It’ll more than do the job until you can get your hands on some hardware.
