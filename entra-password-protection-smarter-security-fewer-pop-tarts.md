---
author: steve@getrubix.com
date: Tue, 04 Feb 2025 13:29:29 +0000
description: '"Alright, let’s be real—passwords are the worst. If you can, ditch them
  altogether and embrace the wonderful world of passwordless authentication. But if
  you’re stuck with passwords for the time being, at least let’s make them a little
  less terrible, shall we?That’s where Microsoft Entra Password Protection comes"'
slug: entra-password-protection-smarter-security-fewer-pop-tarts
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/entra-password-protection-smarter-security-fewer-pop-tarts_thumbnail.jpg
title: Entra Password Protection Smarter Security Fewer Pop-Tarts
---

Alright, let’s be real—passwords are the worst. If you can, ditch them altogether and embrace the wonderful world of passwordless authentication. But if you’re stuck with passwords for the time being, at least let’s make them a little less terrible, shall we?

That’s where **Microsoft Entra Password Protection** comes in. It’s a simple but powerful tool that blocks weak and commonly used passwords, using both Microsoft’s global banned password list and your own custom list. That means no more “Winter2024!” or “CompanyName123.” Plus, if your users have a habit of using something super obvious—like, say, “Rubix” you can block that too.

**Setting Up Password Protection in Your Tenant**

First, let’s grab the [required files](https://www.microsoft.com/en-us/download/details.aspx?id=57071):

-   **AzureADPasswordProtectionDCAgentSetup.msi**
    
-   **AzureADPasswordProtectionProxySetup.exe**
    

Microsoft provides a handy diagram below on how this all works, but let’s break it down step by step.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/5d2c3f41-b0a1-4a62-9f04-8ee684768d65/blog1.png)

**Step 1: Setting Up the Password Protection Proxy**

Microsoft has a few [prerequisites](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-password-ban-bad-on-premises-deploy#microsoft-entra-password-protection-proxy-service), but the general process goes like this:

1.  Install the proxy setup file on a designated server. If you’re in a production environment, it’s best to have multiple proxies for redundancy. (I’m just using one in my test setup.)
    
2.  The installer will not prompt you for a restart, but trust me, just go ahead and reboot. It helps.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/f5007b32-1225-493d-b7b5-6d7591ecd00c/blog2.png)

**Step 2: Register the Proxy Service**

Now that our proxy service server is up and running, we need to register it. Here’s what you need to do:

1.  Import the **AzureADPasswordProtection** module.
    
2.  Make sure the password protection proxy service is running.
    
3.  Run the following command (with Global Admin credentials the first time you do this):
    

```
Register-AzureADPasswordProtectionProxy -AccountUpn 'globaladmin@tenant.domain.com'
```

4. After that, test your setup to confirm everything is working as expected.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1326f84b-4d2c-4fd6-b4b0-8ffd3bb5afd4/bog3.png)

**Step 3: Register Your On-Premises Active Directory**

Next up, we install the **DC Agent** on all domain controllers. It’s a super simple install—just click a button, let it do its thing, and restart when prompted. (Yes, another restart. Just go with it.) Be sure to grab the [prerequisites](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-password-ban-bad-on-premises-deploy#microsoft-entra-password-protection-dc-agent) for this installation.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/8e685e22-fb98-46e2-84c6-81f9c9d2e4d9/blog4.png)

After rebooting, we run similar commands as before to verify everything is properly registered. If all checks pass, we’re good to move on.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/07908292-1c9d-40a8-b77e-9b433aa32d94/blog5.png)

**Configuring Password Protection in Entra**

Now that our on-premises setup is done, let’s configure our policies in Entra:

1.  Head over to **Authentication Methods > Password Protection**.
    
2.  Set up the **Password Protection policy**, including Smart Lockout and your custom banned password list.
    
3.  Want to block specific words? Go for it! In my case, I’m banning anything that resembles “Rubix.” And just for fun, I’m also blocking “poptart” because Steve is obsessed with them. (Sorry, Steve.)
    
4.  Enable **Password Protection on Windows Server AD** to enforce the policies we just created.
    
5.  Set **Mode** to Audit first so you can see what passwords are getting flagged before enforcing it.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/8f22172f-4fb7-4c45-b02a-2538bbeda3b7/blog6.png)

**Putting It to the Test**

Once everything is in place, I tried changing my password to something resembling “poptart” and—boom—rejected faster than a vegetarian at a BBQ competition.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/83b2ed91-7152-4eff-a071-d65a7d048888/blog7.png)

A quick check of the event logs under: **\\Applications and Services Logs\\Microsoft\\AzureADPasswordProtection\\DCAgent\\Admin** confirmed that my new password was rejected due to Azure’s Password Policy. Success!

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/03ebde3e-aea3-4c11-9581-4f16cedf2e83/Screenshot+2025-02-04+090904.jpg)

**Final Thoughts**

Entra Password Protection is an easy way to step up your security game while still keeping things simple for your users. It helps eliminate those embarrassingly weak passwords, protects your organization from brute-force attacks, and lets you have a little fun customizing your banned password list (within reason, of course).

So go ahead—give it a try, ban some bad passwords, and sleep a little better at night knowing your users won’t be securing their accounts with “Password123!” anymore.
