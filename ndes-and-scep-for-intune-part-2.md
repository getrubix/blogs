---
author: steve@getrubix.com  
date: Tue, 11 May 2021 02:41:45 +0000  
description: '"Before we move on to Part 2, there are two tasks I should have included  
  in Part 1.First, we need to give the NDES service account permissions to request  
  and issue certificates.&nbsp; Log into the CA and launch the Certification Authority  
  console.&nbsp; Right click on the CA and"'  
slug: ndes-and-scep-for-intune-part-2  
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/ndes-and-scep-for-intune-part-2_thumbnail.jpg  
title: NDES and SCEP for Intune Part 2  
tags:
  - Intune
  - NDES
  - SCEP
  - Azure AD
  - Certificates
categories:
  - Device Management
  - Enterprise Mobility
---

Before we move on to Part 2, there are two tasks I should have included in [Part 1](https://www.getrubix.com/blog/ndes-and-scep-for-intune-part-1).

First, we need to give the NDES service account permissions to request and issue certificates.  Log into the CA and launch the Certification Authority console.  Right click on the CA and click **Properties**.

![Picture12.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620699811466-SCF6259FJV7A0PXWRYJY/Picture12.png)

On the **Security** tab, add the NDES account and check the boxes for **Issue and Manage Certificates** and **Request Certificates** permissions.

![Picture13.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620699861464-LYFJ5XY108DI9SQEXSA2/Picture13.png)

Head back to the NDES server.  Launch **Computer Management** and add the NDES account to the **IIS_IUSRS** group.

![Picture14.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620699887853-4OB0DAC5MKNS3OQHIBP3/Picture14.png)

All good?  Terrific.  On to Part 2…

### Part 2: IIS Filters, Azure App Proxy, and the Certificate with the external DNS

#### Configure Request Filtering (NDES)

Log into the NDES server and launch the IIS Manager.  Navigate to the **Default Web Site** and select **Request Filtering**.

![Picture15.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620699958991-E5WKMIBMP1KN1XYN8XSW/Picture15.png)

Click **Edit Feature Settings…**

![Whatever it is, the way you tell your story online can make all the difference.](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620699977696-N0JBS252W4076VA2L0ZJ/Picture16.png)

Change the value for **Maximum URL length (Bytes)** and **Maximum query string (Bytes)** to 65534.

![Picture17.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620699999186-ZQ9JN8FKR42YAJQM3T27/Picture17.png)

The requests for certs coming through the Intune connector can get quite lengthy, and we don’t want them getting stuck at the door with the bouncer.

To further solidify those values, open the Registry Editor on the NDES and navigate to **COMPUTER\\HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\HTTP\\Parameters** and add the following DWORD values:

- Name: _MaxFieldLength_  
  - Base: Decimal  
  - Value data: 65534  

- Name: _MaxRequestBytes_  
  - Base: Decimal  
  - Value data: 65534  

![Picture18.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700090337-5XF99PTR016NW5ITIB4P/Picture18.png)

#### Download the Azure App Proxy connector (Azure AD)

Login to Azure AD with global administrator rights at [https://aad.portal.azure.com](https://aad.portal.azure.com) and navigate to **Azure Active Directory -> Application Proxy -> Download connector service.** 

Accept the terms and download.

![Picture19.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700117345-NK7IVQKNI5AQL0TIQDWS/Picture19.png)

#### Install the Azure App Proxy connector (NDES)

On the NDES server, launch the _AADApplicationProxyConnectorInstaller.msi_.  Agree to the terms and click **Install**.

![Picture20.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700155684-9KALMEM3T4PNE98KBZHV/Picture20.png)

When prompted, login with Azure AD global administrator rights.

![Picture21.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700168868-4WLKDO2AOG75GWMQDHK9/Picture21.png)

Assuming you know the password, you should be all set.

![Picture22.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700189035-0P2ROYUWMV0RWHGBWE88/Picture22.png)

Go ahead and close the installer.

#### Add the on-premise application (Azure AD)

Log back into [https://aad.portal.azure.com](https://aad.portal.azure.com) and make your way back to the app proxy.  You should now see the healthy connection as active and pointing to your NDES server.

![Picture23.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700215443-6QYHUS8TT4U0TE5HLZUB/Picture23.png)

Select **+ Configure an app**.

Give the application a friendly name (I chose **SCEP**) and then specify the FQDN of your NDES (e.g., `http://FQDN`).

![Picture24.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700274822-37TXUBKDKEEOZIEKX7NA/Picture24.png)

Azure will automatically concatenate the external URL.  Copy that into a notepad or sticky note because we’re going to need it a few times later. 

Set **Pre-Authentication** to **Passthrough**.  Leave the other values as defaults.  Click **+ Add** when you’re done to save the application.

#### Troubleshooting tip

Be sure the internal URL name does not have any wrong characters or spelling errors, as that will ruin the whole thing.  Like the brilliant mind that I am, I initially entered my internal URL as **http://z0tndes.zerotouch.local** and in reality, it is **http://z0t-ndes.zerotouch.local**. 

That lack of a hyphen sank the whole ship later until I went back and corrected it.

#### Request the NDES certificate (NDES)

We’re going to use the same client/server authentication template we made originally, based off the web server template, to authenticate both the NDES to the CA and for the Intune SCEP connector in Part 3.

On your NDES server, launch MMC and add the local computer certificate snap-in.  Right click on **Personal** and select **All Tasks -> Request New Certificate**. 

![Picture25.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700357447-CVKN1XTZLRNPNQ0FQK16/Picture25.png)

Select **Active Directory Enrollment Policy** and click **Next**.

![Picture26.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700381199-DOEV911VS2WBJ1VLE1QE/Picture26.png)

Find the NDES template you made and click the **More information is required…** link.

![Picture27.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700411860-N6V32QCE1JQFC00CCJME/Picture27.png)

For the **Subject name**, select **Common name** from the drop down.  Add the FQDN of your NDES server as the value and click **Add >**.

For **Alternative name**, select **DNS** from the drop down.  Again, add the FQDN as the value, and then add the external URL of the app proxy from the previous step as the second value.  It should look like this:

![Picture28.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700436434-CS47B6W9Y4YOE2IMOFY2/Picture28.png)

Select **OK**, then check the box next to the template and hit **Enroll**.

![Picture29.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620700453610-TY1DPWYR1JO4LFR105Q8/Picture29.png)

The NDES server will now have the client/server authentication certificate in the **Personal** certificate store.

Alright, I think we’ve all earned a little rest before Part 3.  See you soon.
