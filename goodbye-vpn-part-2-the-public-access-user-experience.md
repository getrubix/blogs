---
author: steve@getrubix.com
date: Sun, 02 Jun 2024 23:08:02 +0000
description: '"I’m often asked questions after I upload my videos, but some of the
  most interesting are:“Hey, Steve; what’s with all the nonsense ranting in the beginning
  of your videos?”“Who do you talk to off to the side of your desk; the Koo-Aid Man,
  Snoopy, yourself?”“Is it a real"'
slug: goodbye-vpn-part-2-the-public-access-user-experience
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/goodbye-vpn-part-2-the-public-access-user-experience_thumbnail.jpg
title: Goodbye VPN Part 2 - The public access user experience
---

I’m often asked questions after I upload my videos, but some of the most interesting are:

-   “Hey, Steve; what’s with all the nonsense ranting in the beginning of your videos?”
    
-   “Who do you talk to off to the side of your desk; the Koo-Aid Man, Snoopy, yourself?”
    
-   “Is it a real basement, or perhaps a very realistic movie set on a sound stage that coincidentally has sub-par audio and video quality compared to other big-budget Hollywood projects?”
    
-   “This is a Best Buy- why are you trying to return those pants?”
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/190d7931-8569-4095-a1e0-ba194d00a23c/PXL_20240528_172145751.MP.jpg)

Looks real, but you never know…

The point is, engaging with people is fun.

On that note, let’s move on by looking at the Entra Private Access user experience with the Global Secure Access Client on Windows.

End-user setup
--------------

For our “testing”, we took a brand-new Windows 11 PC and went through the standard Autopilot setup. A few things to note about this setup.

-   PC will be Entra ID joined only (cloud native)
    
-   Our user, _Morty Smith_ is synced from on-premises Active Directory and has access to file shares on the _RBXDV-DC-01_ server
    
-   We are 100% remote, with absolutely no line-of-sight to our domain
    

We start by powering on the PC and going through Autopilot sign-in.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/481c14fe-6eec-4eb6-bcd6-2e2f0110bd21/Screenshot+2024-06-02+at+12.06.21%E2%80%AFPM.png)

When the provisioning is complete, you can see we’re signed in as Morty Smith, and the Intune Company Portal app has been deployed to our machine.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/11d42d73-f936-4eb1-87e7-d49d18f88e80/Screenshot+2024-06-02+at+12.23.03%E2%80%AFPM.png)

After clicking on the Company Portal, we see the Global Secure Access Client app. Let’s select it and click **Install**.

![Screenshot 2024-06-02 at 12.22.25 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1717368622157-I48V6M2605EZHUJJ4B7C/Screenshot+2024-06-02+at+12.22.25%E2%80%AFPM.png)

![Screenshot 2024-06-02 at 12.22.25 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1717368622157-I48V6M2605EZHUJJ4B7C/Screenshot+2024-06-02+at+12.22.25%E2%80%AFPM.png)

![Screenshot 2024-06-02 at 12.23.16 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1717368622095-MS6WGWNQJPLVL92YJ31A/Screenshot+2024-06-02+at+12.23.16%E2%80%AFPM.png)

![Screenshot 2024-06-02 at 12.23.16 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1717368622095-MS6WGWNQJPLVL92YJ31A/Screenshot+2024-06-02+at+12.23.16%E2%80%AFPM.png)

#block-yui\_3\_17\_2\_1\_1717368278924\_18084 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -20px; } #block-yui\_3\_17\_2\_1\_1717368278924\_18084 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 20px; margin-bottom: 20px; }

For this demo, we made the app _available_ from Intune, but there’s no reason why we can’t push the client as required, so it is automatically present.

Connected like magic
--------------------

Once the installation is complete, the Global Secure Access Client prompts Morty to authenticate. This is where you have the opportunity to enforce MFA with conditional access if you’d like (more on that in a future post).

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/3ae54292-8d08-42ec-895a-9fec449c6f6b/Screenshot+2024-06-02+at+12.25.20%E2%80%AFPM.png)

After the sign-in, Morty can connect to his usual file shares on the domain server as if he was on the network.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/3354b645-2eb7-463a-83fb-2a7796e85c65/Screenshot+2024-06-02+at+12.26.36%E2%80%AFPM.png)

Aaaaaaaaand done!
-----------------

That’s it! Nothing else needs to be done on Morty’s part, and his device is not actually on the corporate network, nor is there a VPN client running. If you want to see the traffic in action, you can see it in the Entra portal.

-   Sign in to [entra.microsoft.com](http://entra.microsoft.com/) and navigate to **Global Secure Access** -> **Monitor** -> **Traffic logs**
    
-   You can see the session info including source IP address, username, and the destination FQDN
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/2bfd8467-bfef-41c4-a24e-8b5ac466fdcf/Screenshot+2024-06-02+at+5.59.58%E2%80%AFPM.png)

While in this example we’re using this for file shares, we can extend this to Remote Desktop, mapped drives, and on-premises application authentication.

In the next part, we’ll discuss using conditional access, MFA, and even Cloud PKI to protect our private resources.
