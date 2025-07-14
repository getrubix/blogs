---
author: steve@getrubix.com
date: Sat, 04 Jan 2020 19:55:00 +0000
description: '""'
slug: on-premise-access-from-azure-with-windows-hello-for-business
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/on-premise-access-from-azure-with-windows-hello-for-business_thumbnail.jpg
title: On-premise access from Azure with Windows Hello for Business
---

![2020-01-03-19_27_39-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105147421-8M26XUO9HH8N73HVIIXO/2020-01-03-19_27_39-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png)

I have an upcoming series of posts coming out soon all about Autopilot hybrid join. In fact, this was supposed to be the first on the topic. But I’m walking a delicate balance between forcing people into modern IT and appeasing the needs of those folks who legitimately aren’t ready for cloud-only. So to that point, we’re going to talk about Windows Hello for Business and why it gives you one less reason to domain join (or hybrid join) a Windows 10 PC.

There is a common misconception about Hello; that it is solely the technology of unlocking a laptop with your face or thumbprint. While that’s a solid pitch to consumers at retail, it is a gross oversimplification. Here is the actual definition, as per [Microsoft’s documentation](https://docs.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-identity-verification):

“_Windows Hello for Business replaces passwords with strong two-factor authentication on PCs and mobile devices. This authentication consists of a new type of user credential that is tied to a device and uses a biometric or PIN.”_

For our purposes, this modern method of authentication can be used to access on-premises resource from an Azure AD joined machine. This is the most requested feature from customers when building an Autopilot deployment, mostly for mapped drives and network printers.

For this example, I’m going to show you a scenario from my test lab. The lab environment consists of:

-   2016 Domain Controllers
    
-   Local AD
    
-   Azure AD Premium
    
-   Azure MFA (Multi-factor authentication)
    
-   Microsoft Endpoint Management (Intune)
    
-   Windows 10 1903 clients
    

While access to file shares from Azure joined machines has been _technically_ possible, the user experience hasn’t been great. That’s because the PC does not exist on the local domain, therefore it cannot pass the credentials back through automatically when authenticating to the specified share.

Even though the user should have access, we see a disruptive prompt for credentials:

![2020-01-04-08_37_26-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105190384-MVW9CYMOXKTJ6V9PYI46/2020-01-04-08_37_26-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png)

What do I have to do?
---------------------

Hello for Business changes this by replacing the password all together, at least for the PC. Here is a super high-level outline of the flow:

-   PC joins to Azure AD with Autopilot
    
-   User creates PIN after MFA is configured.
    
-   Windows Hello generates a unique device key that is trusted by Azure AD
    
-   Root certificate from an on-premise domain controller is deployed to the client via Intune.
    
-   On-premises AD can accept Hello authentication when a request is made to the resource from an Azure AD joined machine
    

Disclaimer- there are about one billion components to everything I just described, so please take it as an outline and not a bible. There are also many ways Hello can be implemented and for varying reasons, which all involve many steps- all of which vary based on the infrastructure. For instance, one of the key factors is the identity schema in Active Directory. My servers are all 2016 and support the schema level required for Hello for Business. For all the prerequisites and painful step by step instructions, read [all of this](https://docs.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-planning-guide).

On-premise prep
---------------

So after all of that heavy research, I created a simple certificate infrastructure within my domain. That way my domain controllers could use KDC (Key Distribution Center) authentication criteria for incoming requests from Azure AD joined machines. Here is what had to be configured:

-   **CRL (_Certificate Revocation List_) Distribution Point**: I had to install the Web Server role on my domain controller with the certificate authority service to host the CRL list on an http site, so it is reachable on the domain.
    
-   **Domain Controller certificate using the Kerberos Authentication template**: That one was a lot of words. Using the Certificate Authority console on the server with the CA (_Certificate Authority_) role, I created a root domain controller cert from the Kerberos Authentication certificate template. This is then issued with group policy to all domain controllers.
    
-   **Export Root certificate**: Once the Kerberos Authentication certificate is deployed to the domain controllers, copy their root cert to a file that will be used for deployment through Intune to the Windows 10 clients.
    

Step-by-step instructions for these pieces can be [found here](https://docs.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-guide). I strongly recommend you go through them.

Can we get back to the Azure part?
----------------------------------

Now that the local domain is properly configured, we can enable Intune to deploy Hello for Business. There are three things that have to happen when logging into [https://devicemanagement.microsoft.com](https://devicemanagement.microsoft.com/):

-   Enable Windows Hello for Business enabled for the tenant. This is done by navigating to **Devices -> Enroll devices -> Windows Hello for Business**
    

![2020-01-03-18_39_21-enroll-devices-windows-enrollment-microsoft-endpoint-manager-admin-center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105222640-7R1EXIHM63JJDWP1J9X4/2020-01-03-18_39_21-enroll-devices-windows-enrollment-microsoft-endpoint-manager-admin-center.png)

-   Next, configure the individual _Identity protection_ configuration profile and assign it to users and devices that you want leveraging Hello. **Devices -> Configuration profiles -> + Create Profile**. Select **Identity protection** as the profile type.
    

![2020-01-03-18_44_50-trusted-certificate-microsoft-endpoint-manager-admin-center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105247324-J0V3SCDNYZCO28E1404L/2020-01-03-18_44_50-trusted-certificate-microsoft-endpoint-manager-admin-center.png)

-   Finally, configure a profile with the exported root certificate from the domain. Go to **Devices -> Configuration profiles -> + Create Profile**. Choose **Trusted certificate** as the type. Upload your cert and make sure it’s set to go to **Computer certificate store – Root**.
    

![2020-01-03-18_49_58-windows-hello-for-business-microsoft-endpoint-manager-admin-center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105275111-3CTQ6UUJYUWPUQNOR594/2020-01-03-18_49_58-windows-hello-for-business-microsoft-endpoint-manager-admin-center.png)

When the user goes through joins the PC to Azure via Autopilot for the first time, they’re asked to either verify or setup their MFA.

![2020-01-03-19_28_19-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105297786-ACHTOMCTLW63284R7M4C/2020-01-03-19_28_19-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png)

Once configured, they’re prompted for a device PIN (_and optionally biometric authentication_) before being brought to the desktop. And that’s it!

Test by logging into a mapped drive or just navigating directly to a network files share. If everything worked, there should be absolutely no prompt for credentials:

![2020-01-04-08_55_10-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1581105322885-H9KAIE243YW0RUWDU20X/2020-01-04-08_55_10-rdg.batcave.local-remote-desktop-connection-manager-v2.7.png)

As always, feel free to hit me up with questions and stay tuned for a massive saga about hybrid join coming up!
