---
author: steve@getrubix.com
date: Tue, 28 Jan 2025 18:15:54 +0000
description: '"Hey there! Today, we''re diving into the wonderful world of passkeys.
  Don''t worry, this isn''t rocket science – my mission is to show you how ridiculously
  easy it is to get passkeys up and running in your tenant. Sure, standard MFA is
  a solid starting point, but let''s"'
slug: lets-set-up-passkeys-the-easy-secure-way
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/lets-set-up-passkeys-the-easy-secure-way_thumbnail.jpg
title: Lets Set Up Passkeys The Easy Secure Way
---

Hey there! Today, we're diving into the wonderful world of **passkeys**. Don't worry, this isn't rocket science – my mission is to show you how ridiculously easy it is to get passkeys up and running in your tenant. Sure, standard MFA is a solid starting point, but let's face it - it’s not exactly the finish line of cybersecurity greatness. 

So, what are passkeys, you ask? Think of them as passwordless VIP pass into your apps. Instead of juggling passwords, passkeys use cryptographic key pairs to get you logged in safely and securely. Your private key is on your device, while the public key hangs out with the service. Together, they create a phishing-resistant fortress of login magic. 

### **Step 1: Turning on Passkeys** 

First things first, we’ve got to enable passkeys for users. In Entra, just head over to **Protection > Authentication Methods** and select Passkeys. Easy, right? If you’d like to roll it out gradually, start with a pilot group—like my creatively named **Passkey Pilot Group**. 

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/f96fe9d7-81a0-484f-9584-f3f2cbd019a5/passkey1.png)

### **Step 2: Configure Those Options**

Time to tweak a few settings! Head over to **Configure**, and let’s break these down:

-   **Allow self-service setup:** Let users set up their own passkeys. Saves you from playing tech support all day.
    
-   **Enforce attestation:** Want only trusted devices in the mix? This setting ensures devices prove their authenticity.
    
-   **Enforce key restrictions:** Want a little more control? This lets you decide which FIDO2 keys are allowed or blocked.
    
-   **Restrict specific keys:** Choose whether certain keys get the red carpet treatment or a big ol’ “Access Denied.”
    
-   **Microsoft Authenticator:** Automatically handles AAGUIDs (don’t worry, just think of it as tech magic) for iOS and Android Authenticator apps.
    

For this setup, we’re using Microsoft Authenticator, but you can totally swap in FIDO keys if that’s your vibe. Pro tip: This is [handy link](https://support.yubico.com/hc/en-us/articles/360016648959-YubiKey-Hardware-FIDO2-AAGUIDs) to identify AAGUIDs for other popular keys.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/f8409263-84aa-4f59-b430-3c8baa7a83b3/passkey2.png)

### **Step 3: User Setup**

Now, the fun part! Direct your users to [aka.ms/mfasetup](http://aka.ms/mfasetup) and tell them to click **Add sign-in method**. Select **Passkey** and follow the prompts. It’s as simple as opening the Microsoft Authenticator app, creating the passkey, and they’re good to go.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/8a2b1b05-0b5a-4d53-8f38-e932ac528060/passkey3.png)

Select **Passkey** and follow the on-screen instructions.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7c19f1fe-6514-4d77-8b8d-489ccd20316b/passkey4.png)

Part of this next step will direct you to open the Microsoft Authenticator application on your device. Create a Passkey in the Authenticator application.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/02bd0d86-907e-45cd-b9d4-e609c9acbae5/passkey5.png)

The user will be prompted to **Sign In**.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/80468131-8c59-4380-b733-4f5ad83c8a57/passkey6.png)

Once the passkey is created, select **Done.**

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/d253a4c8-363e-4b6e-8e27-1a87bc576339/passkey7.png)

Back on the users PC, they should receive a notification that their passkey is now created.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/77360711-565e-4061-a318-a1676031f230/passkey8.png)

### **Pro Tip for Admins**

Here’s where things get serious (but not _too_ serious): Make sure your admins are using passkeys, too. You can quickly lock this down with Conditional Access Policies to require phishing-resistant MFA for all admin activities.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e6032420-73b9-4f35-bf1e-ff6f7f67eeab/passkey9.png)

### **And That’s It!**

See? Just a few quick steps to make your users more secure and less stressed about passwords. Go ahead and give it a try, you’ll feel like a cybersecurity superhero in no time.
