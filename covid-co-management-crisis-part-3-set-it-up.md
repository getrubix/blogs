---
author: steve@getrubix.com
date: Mon, 07 Sep 2020 17:42:25 +0000
description: 'If you read Part 2 in this series, then you know what happens now.
  Here is a rant-free guide to setting up the Cloud Management Gateway for SCCM.'
slug: covid-co-management-crisis-part-3-set-it-up
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/covid-co-management-crisis-part-3-set-it-up_thumbnail.jpg
title: Covid Co-Management Crisis Part 3 Set it up
---

If you read [Part 2](https://www.getrubix.com/blog/covid-co-management-crisis-part-2-cloud-management-gateway) in this series, then you know what happens now. Here is a rant-free guide to setting up the Cloud Management Gateway for SCCM.

## Steps in this guide
---

1. [Verify a unique URL DNS name](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#verify)
    
2. [Certification preparation](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#cert)
    
3. [Azure service integration to SCCM](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#azure)
    
4. [Deploy Cloud Management Gateway](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#deploy)
    
5. [Verify resource creation in Azure](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#resource)
    
6. [Configure connection point role](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#configure)
    
7. [Site system and management point settings](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#site)
    
8. [Enable software update point](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#enable)
    
9. [Primary site settings](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#primary)
    
10. [Client settings](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#client)
    
11. [Deploy client to Intune](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#intune)
    

>Cloud Management gateway clients can be authenticated via either PKI certificates or Azure AD Authentication. We will leverage the latter. This works with both Azure AD Joined or Hybrid-Azure AD Joined scenarios_

## Prerequisites
---

The following prerequisites are required during implementation

-   Azure AD Connect enabled for Hybrid Azure-AD Join
    
-   SCCM Current Branch version 1910
    
-   Microsoft.ClassicCompute resource provider registered within Azure subscriptions
    
-   Microsoft.Storage resource provider registered within Azure subscriptions
    

## Verify a unique URL DNS name
---

>This will be done for both the Cloud.Compute resource and the Storage resource

1. Login into [https://portal.azure.com](https://portal.azure.com) with admin credentials
    
2. Search the ‘resources’ field for **Cloud service**
    

![1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599490422126-GXEZ4HGSBXNQMBG5WP14/1.png)

3. In the ‘DNS name’ field, enter your desired name.  A green check will populate if the name is available.  No further configuration is needed; we just needed to validate the name availability.  Close out of this menu and return to the main Azure homepage
    

![2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599490501312-SIG8JKT3E5IPIC5HVIID/2.png)

4. Search the ‘resources’ field for **Storage accounts**
    

![3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599490567263-K2E244WVXC7W9LK1J9GV/3.png)

5. From the Storage accounts menu, click **\+ Add**
    
6. In the ‘Storage account name’ field, enter your desired name.  If a green check mark populates, it is available.  Again, no further configuration needed.  Close out of this menu
    

![4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599490649838-3X4P3JI900LGY0588KF1/4.png)

## Certificate preparation
---

>Take note of both names tested above.  Do not proceed with any configuration- we will need those names later.

1. Log into your Certificate Authority server with administrator credentials and launch the **Certification Authority** console.
    
2. Right-click on **Certificate Templates** and click ‘Manage’
    

![1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599490960472-UH8KMEEVPRGXUNOV4SPK/1.png)

3. Right-click on the **Web Server** template and click ‘Duplicate Template’
    

![2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599491129118-T2R4VNBNU82W6Q73RVWE/2.png)

4. In the ‘General’ tab, enter a display name.
    

![3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599491172183-LXYFEIZ5WS36DVZB88MG/3.png)

5. In the ‘Resource Handling’ tab, select the box for **Allow private key to be exported**.
    

![4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599491212471-HPRZ09DZC47BK4PGZ5KY/4.png)

6. In the ‘Security’ tab, add the group containing your SCCM site server.  Check the box for **Read** and **Enroll** permissions.  Click **OK** to create the certificate template.
    

![5.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599491251992-6HOWBXWJKHD51TSYU82M/5.png)

7. Launch the **Certification Authority** console again. 
    
8. Right click on **Certificate Templates >** **New > Certificate Template to Issue**
    

![8.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599491499881-I8D9MW9WJL15U2M0KAEL/8.png)

9. Select the template we’ve just created and click **OK**
    

![Screen Shot 2020-09-07 at 11.46.52 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599493654315-80YL0NSOF6WBP2COBKFP/Screen+Shot+2020-09-07+at+11.46.52+AM.png)

Next, we will request the newly created certificate on our SCCM primary site server.

>It is recommended that you reboot the SCCM server prior to requesting a new certificate.  This will allow the SCCM server to refresh the computer authentication token with the CA._

1. Log into the SCCM primary site server with admin credentials.
    
2. Launch **MMC** and add the snap in for **Certificates > Local Computer**.
    
3. Navigate to **Certificates (Local Computer) > Personal > Certificates**.
    
4. Right-click on **Certificates** and select **All Tasks > Request New Certificate**
    

![Screen Shot 2020-09-07 at 11.48.16 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599493708628-KST6JC88ATWWAWMZ5XYA/Screen+Shot+2020-09-07+at+11.48.16+AM.png)

5. Select **Next** twice on the Certificate Enrollment window.
    

![Screen Shot 2020-09-07 at 12.10.40 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495202692-TRCL13OZ68P2LU657CAD/Screen+Shot+2020-09-07+at+12.10.40+PM.png)

6. Select the certificate template we issued and click the ‘more information’ link.
    

![Screen Shot 2020-09-07 at 12.11.34 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495228803-D77HJT3IYL52DYX4KNMC/Screen+Shot+2020-09-07+at+12.11.34+PM.png)

7. Choose **Common Name** from the ‘Subject Name’ drop-down
    
8. For the value, enter the unique name we verified was available in [Step 1](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#_Verify_a_unique).
    

![Screen Shot 2020-09-07 at 11.27.13 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495258880-HJCZXUQFFBCXYOAONLKA/Screen+Shot+2020-09-07+at+11.27.13+AM.png)

9. Click **Enroll**
    

![Screen Shot 2020-09-07 at 11.27.18 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495284598-V5WEZDEZSMEIUF8YWDX8/Screen+Shot+2020-09-07+at+11.27.18+AM.png)

10. After the certificate is enrolled, click **Finish**.
    

![Screen Shot 2020-09-07 at 11.27.26 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495321177-2M24S2Y25VFM0SIT1TGB/Screen+Shot+2020-09-07+at+11.27.26+AM.png)

Now we will export the private key from the certificate that requested.

11. On the SCCM site server, launch **MMC** and add the local computer certificate snap in again.
    
12. Navigate to **Certificates (Local Computer) > Personal > Certificates.**
    
13. Right-click on our new certificate and select **All Tasks > Export**.
    

![14.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599492182311-L1BT53D7E3TB6CTUSCOU/14.png)

14. Click **Next** on the Certificate Export Wizard and select **Yes, export the private key**
    

![15.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495361154-RHAPPE32PA1RUC7FYC9X/15.png)

15. Verify the following options on the ‘Export File Format’ page- click **Next**
    

![16.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495386067-ULL7ERSBWIK420YWHIVU/16.png)

16. Enter a password to protect your private key and click **Next**
    

![Screen Shot 2020-09-07 at 12.12.51 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495413904-3V9B5BBLSASSUL4PDMDV/Screen+Shot+2020-09-07+at+12.12.51+PM.png)

17. Choose a location and save the .PFX file.
    

![18.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495458361-VMWF6QK05LT9M5FJFAD8/18.png)

## Azure service integration to SCCM
---

1.  Log into SCCM console and navigate to **Administration > Cloud Services > Azure Services**
    
2. Click **Configure Azure Services** in the ribbon.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495821778-1NEEOSWRN5SJ4QCQV36D/image-asset.png)

3. In the ‘Configure Azure Services’, provide a name and select **Cloud Management**.
    

![Screen Shot 2020-09-07 at 12.22.09 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495854790-K3Z9RYZZOUCD455W501H/Screen+Shot+2020-09-07+at+12.22.09+PM.png)

4. We need to provide names for the **Web app** and **Native Client app**.  Once provided, sign in to your Azure tenant with Global Administrator credentials. 
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495877221-TEBNQ44MMFZF1C82RO2Q/image-asset.png)

5. Once finished, log into [https://portal.azure.com](https://portal.azure.com) and navigate to **Azure Active Directory > Enterprise applications**.  You will see the two new entries.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599495903147-NY056M128H9SM1MUB5ER/image-asset.png)

## Deploy Cloud Management Gateway
---

1. Log into SCCM console and navigate to **Administration > Cloud Services > Cloud Management Gateway**
    
2. Click on **Create Cloud Management Gateway** in the ribbon.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498175448-1XRMZY0KP7TY3PMKU7HW/image-asset.png)

3. Click **Sign In** and proceed with your Azure Global Admin account.  Select a subscription from the drop down.  The **Azure AD app name** should automatically populate from [Step 3](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#azure).  Click **Next**.
    

![Screen Shot 2020-09-07 at 1.02.16 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498213340-BSCZ5LKXCC4O8YERMGYM/Screen+Shot+2020-09-07+at+1.02.16+PM.png)

4. On the next page, click **Browse** to upload the .PFX cert that was created in [Step 2.](https://www.getrubix.com/blog/covid-co-management-crisis-part-3-set-it-up/#cert)  Enter the password when prompted.  Choose the region you want the Azure service to run in. 
    
5. Create a new **Resource Group**. 
    
6. For the **VM Instance** number, keep in mind that one VM per CMG can support up to 6000 clients, 2000 of them simultaneously.  See more information on scaling [here](https://docs.microsoft.com/en-us/configmgr/core/clients/manage/cmg/plan-cloud-management-gateway#performance-and-scale).
    
7. Make to uncheck the box for **Verify Client Certificate Revocation**.  
    Lastly, make sure the box is checked for **Allow CMG to function as a cloud distribution point and serve content from Azure storage**.
    

![Screen Shot 2020-09-07 at 1.02.27 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498271053-T8QNTOZ34R72WDOZGSA0/Screen+Shot+2020-09-07+at+1.02.27+PM.png)

8. Click **Next**.  Allow several minutes for the service to be provisioned. 
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498350941-BZVDMB4NFV0OVRDL28HU/image-asset.png)

Status of the CMG can be confirmed in both Azure and SCCM.

## Verify resource creation in Azure
---

1. Log into [https://portal.azure.com](https://portal.azure.com) and select **All resources**.  You will see the two new resources (cloud service and storage account) that make up the CMG
    

![Screen Shot 2020-09-07 at 1.07.02 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498470109-OZX901YB6CQ8SXJW6ILD/Screen+Shot+2020-09-07+at+1.07.02+PM.png)

2. Click on the cloud service to see more information.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498491953-NG1X0HYS4V621QOYPAQW/image-asset.png)

## Configure connection point role
---

1. In SCCM, navigate to **Administration > Site Configuration > Servers and Site System Roles**
    
2. Right-click on the primary site server and select **Add Site System Roles**
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498637353-TWONEV7K57L6SQXUIJ8C/image-asset.png)

3. Click Next through the wizard to reach the **Specify roles for this server** window.  Check the box for **Cloud management gateway connection point** and click **Next**.
    

![Screen Shot 2020-09-07 at 1.09.04 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498711491-AVWVO2DUDR7ICHVSIPNT/Screen+Shot+2020-09-07+at+1.09.04+PM.png)

4. The role should now be listed under **Site System Roles** for that server
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498742036-9Z3D24482TNIBPAHPVPH/image-asset.png)

## Site system and management point settings
---

1. On the **Servers and Site System Roles** page, select the primary site.  Right-click on **Management point**.
    

![Screen Shot 2020-09-07 at 1.12.37 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498814183-7VTXVV3CLR7P7V5Y7U65/Screen+Shot+2020-09-07+at+1.12.37+PM.png)

2. Make sure the box is checked for **Allow Configuration Manager cloud management gateway traffic**. 
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498852564-EUSRYSAOLRBLEP2XR7NJ/image-asset.png)

>In my instance, I had the dropdown above set to **Allow intranet and Internet connections**.  If you decide to use a dedicated management point for internet clients, that is an option.

3. Click **OK** to complete the management point setup.
    

## Enable software update point
---

1. On the **Servers and Site System Roles** page, select the primary site.  Right-click on **Software update point**.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599498965136-V0WC2WFJTKWHU23R3E01/image-asset.png)

2. Make sure the box is checked for **Allow Configuration Manager cloud management gateway traffic**. 
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599499028311-2S6RUOUQ6SBY3IK7DQAC/image-asset.png)

## Primary site settings
---

3. Navigate to **Administration > Site Configuration > Sites**.
    
4. Right-click on the primary site and select **Properties**
    

![Screen Shot 2020-09-07 at 1.17.31 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599499144340-0KBMW7CW2RAO8REG0GA7/Screen+Shot+2020-09-07+at+1.17.31+PM.png)

5. Select the ‘Communication Security’ tab.  Check the box for **User Configuration Manager-generated certificates for HTTP site systems**.
    
6. Ensure the box for **Use PKI client certificate (client authentication capability) when available** is unchecked.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599499218014-KTGD0BX2OZREZJ2NSD7H/image-asset.png)

7. Click **OK**
    

## Client settings
---

1. Navigate to **Administration > Site Configuration > Client Settings**.  Right-click on current client and choose **Properties**
    

![Screen Shot 2020-09-07 at 1.20.58 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599499319953-NZMU0ZVL5ZZYFHG714E2/Screen+Shot+2020-09-07+at+1.20.58+PM.png)

2. Select **Cloud services**.  Ensure that **Allow access to cloud distribution point** and **Enable clients to use a cloud management gateway** are both set to **Yes**.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599499361601-1XBVMNP3AJXY81HMD6LA/image-asset.png)

## Deploy client to Intune
---

1. Navigate to **Administration > Cloud Services > Co-management**.  Click **Configure co-management** in the ribbon
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599499513317-OJ7QD2CKG5WSXM6I6H0S/image-asset.png)

2. Copy the command line arguments from the properties window.  We will use these to deploy the client through Intune.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599499538808-GLMHM9258S5EX7D3XEU8/image-asset.png)

3. The .MSI client should be located in \\\\<SCCM-SITE>\\Client\\**ccmsetup.msi**
    

![Screen Shot 2020-09-07 at 1.24.23 PM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1599499572466-H6YW5VXIHDRSSAFX007N/Screen+Shot+2020-09-07+at+1.24.23+PM.png)

If you’ve gotten this far, I would assume you don’t need to know how to wrap an application for Intune deployment. But, just in case, [here you go](https://www.getrubix.com/blog/app-answers-yes-intune-can-do-it).

Okay- I’m pretty tired after that. Good luck with the setup.
