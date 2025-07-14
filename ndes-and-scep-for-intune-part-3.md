---
author: steve@getrubix.com
date: Tue, 11 May 2021 20:28:22 +0000
description: "\"Let’s start with some follow up before moving on.&nbsp; We need to
  set the SPN (Service Principal Name) for the NDES account.Log into your NDES server
  and open an elevated CMD prompt.&nbsp; Type the following:setspn -s http/&lt;NDES-FQDN&gt;
  domainName\NDESaccountNameMine looks like this:\""
slug: ndes-and-scep-for-intune-part-3
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/ndes-and-scep-for-intune-part-3_thumbnail.jpg
title: NDES and SCEP for Intune Part 3
tags:
  - Intune
  - NDES
  - Certificate Connector
  - SCEP
categories:
  - Endpoint Management
  - Security
---

**Let’s start with some follow up before moving on.**  We need to set the SPN (Service Principal Name) for the NDES account.

Log into your NDES server and open an elevated CMD prompt.  Type the following:

```
setspn -s http/<NDES-FQDN> domainName\NDESaccountName
```

### Close the CMD prompt

Mine looks like this:

![Picture2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620763636537-05HE68NY0HH89PZ3A8LU/Picture2.png)

Close the CMD prompt when it completes.  Moving on…

### The Binding (NDES)

Now that we have the NDES client/server authentication cert issued to our NDES, we need to bind it to the IIS default site.  Log into the NDES server and launch the IIS Manager.  Navigate to the “Default Web Site” and on the far right, click **Edit Site -> Bindings**.

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620763673492-49459HXCEF1H7X81IOZW/Picture3.png)

Click **Add** on the “Site Bindings” menu.

Make the following changes:

**Type**: https

**Port**: 443

**IP address**: All Unassigned

**Host name**: leave blank

**SSL certificate**: choose the certificate we just issued to the NDES at the end of [**Part 2**](https://www.getrubix.com/blog/ndes-and-scep-for-intune-part-2)

Click **OK**, and close the IIS manager.

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620763756065-PDZULHOT1G1IYBH094D0/3.png)

### Templates in the registry (NDES)

We must configure the registry so that NDES knows which cert template to use when a request comes in from the connector.  This can be defined specially by the purpose of the cert, but to be safe, we’re going to configure all three available options.

On the NDES server, open the Registry Editor and navigate to the following path:

```
Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography\MSCEP
```

There are three values:

```
EncryptionTemplate
```
    
```
GeneralPurposeTemplate
```
    
```
SignatureTemplate
```
    

Edit each one to be the name of your NDES client cert template.

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620763793804-FCT05O7CQ0Y23ZLI49ZB/Picture4.png)

### Download the SCEP connector (Intune)

Log into [https://endpoint.microsoft.com](https://endpoint.microsoft.com) and navigate to **Tenant administration -> Connectors and tokens -> Certificate connectors**.  Click **+Add** and proceed to download the SCEP connector software.

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620764323661-3AAMTMHMWWZ1I7JWQNQ6/Picture5.png)

### Install the connector (NDES)

Copy the NDESConnectorSetup.exe over to your NDES server and launch the installer.  Click **Next** when the setup starts.

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620764370874-R0UYEODMNLYL589L28BP/Picture6.png)

Accept the terms and click **Next**.

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620764407737-AWUQS50DCLJEVVQ6TF35/Picture7.png)

On the Installation options menu, select **SCEP and PFX Profile Distribution**.  Click **Next**.

If prompted to select a certificate, choose the Web Server template we made originally used for client/server authentication.  The same one we issued to the NDES.

When the install is complete, check the box for **Launch Intune Connector** and click **Finish.**

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620764448710-1YRQ0G8Y6T7RZALE6YM6/Picture8.png)

Click **Sign In** to authenticate to Azure.

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620764485511-INMBYP97ABD7ZZ52EFKW/Picture9.png)

Sign into Azure with global administrator credentials.

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620764510475-CNGOYENTLZXO2EHGJD61/Picture10.png)

Once enrolled, click the “Advanced” tab and select **Specify different account username and password**.  Enter the NDES service account credentials.

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620764532208-Y6RA8LN8XK8N11AC14LM/Picture11.png)

Congratulations.  You’ve installed the Intune Certificate connector.  To validate, navigate back to the “Certificate Connectors” section of Intune.  You should see the healthy connector with an “Active” status.

As a great, New Jersey man once said, **“Ooh, we’re half way there…”** (well technically, ¾ there).
