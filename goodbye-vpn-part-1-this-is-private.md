---
title: "Goodbye VPN Part 1 - This is private"
slug: "/blog/goodbye-vpn-part-1-this-is-private"
date: "Tue, 28 May 2024 03:18:01 +0000"
author: "stevew1015@gmail.com"
description: "Recently, I released a guided walkthrough of setting up Entra Private Access for cloud native PCs connecting to on-premises file shares. In addition to a tremendously positive response (thank you), I also received many requests for a written guide. It then got me thinking... could some of my videos benefit from a written companion?"
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/goodbye-vpn-part-1-this-is-private_thumbnail.jpg
---

Recently, I released a [guided walkthrough of setting up Entra Private Access](https://youtu.be/uIjGDUhvKvY) for cloud native PCs connecting to on-premises file shares. In addition to a tremendously positive response (thank you), I also received many requests for a written guide. It then got me thinking... "could some of my videos benefit from a written companion?"

Instead of sitting around thinking about the pros and cons, I decided it doesn't matter and began to make one anyway. Perhaps it will catch on.

What is Entra Private Access
----------------------------

According to Microsoft, Entra Private access is defined like this:

_\> Microsoft Entra Private Access unlocks the ability to specify the fully qualified domain names (FQDNs) and IP addresses that you consider private or internal, so you can manage how your organization accesses them. With Private Access, you can modernize how your organization's users access private apps and resources. Remote workers don't need to use a VPN to access these resources if they have the Global Secure Access Client installed. The client quietly and seamlessly connects them to the resources they need._

In other words, it is a proxy-based system that allows a remote endpoint with the **Global Secure Access Client** to communicate with an on-premises resource through a proxy-connector. It supports common protocols like HTTP, RDP, and SMB and uses Entra Identity for authentication. This is powerful security boon as you can mix in conditional access policy, identity protection, and Defender for Identity telemetry; all adhering to a Zero Trust framework.

Requirements
------------

This is still an 'in-preview' product, which means like all Microsoft previews, it is very much subject to change in terms of features and, more important, pricing.

### License

As of right now, the preview is included in the Entra ID P1 or P2 license, which should allow for most organizations to be able to get this up and running.

### Server

The application proxy connector needs to be installed on a Windows Server. Here are those requirements:

-   2012 R2 or higher
-   minimum .NET version 4.7.1

### Roles

In order to follow this setup guide, you'll need to be either a **Global Administrator** or have all of the following role assignments:

-   Global Secure Access Administrator
-   Application Administrator
-   Conditional Access Administrator

Setup guide
-----------

### Turning it on

-   Start by logging into Microsoft Entra admin center at **entra.microsoft.com** and navigate to **Global Secure Access (Preview)** -> **Connect** -> **Traffic forwarding**
-   In the **Traffic forwarding** menu, enable **Private access profile** ![alt text](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/0e35ae81-3696-47af-947d-072c7c292444/Screenshot+2024-05-27+214945.png)
-   Next to **User and group assignments**, click _View_
-   These are the users the profile will apply to. Either choose specific users and groups or select _Assign to all users_

### Making an app

With private access, an _Application Segment_ is an on premises resource you want to provide access to via the proxy connector.

-   In the main blade, click **Applications** -> **Quick Access** ![alt text](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/bc4835a5-3757-438b-a6ed-809183dfafcb/Screenshot+2024-05-27+215937.png)
-   Under **Application Segment**, click **\+ Add Quick Access application segment**
-   On the **Create application segment** page, define the following settings:
    -   **Destination type**: _Fully qualified domain name_
    -   **Fully qualified domain name**: _Name of the server your file share is on_
    -   **Ports**: _Port you're providing access on_ ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/c1730335-06bc-451b-b266-f1e7559cab41/Screenshot+2024-05-27+220447.png)
        
        > Remember, because we're choosing these setting to provide access to an SMB file share. Other components will require different options such as RDP or IIS.
        

Click **Apply** when finished.

The application is now visible in **Enterprise applications**, just like any other Entra app. Let's add our users.

-   Select the name of your app from **Enterprise applications** ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/ae00f2ed-0331-4002-ba87-a8f473fb4585/Screenshot+2024-05-27+220511.png)
-   On the application page, navigate to **Users and groups** and click **\+ Add user/group** ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/297e1608-c3dd-4d10-ae6b-cb3952570c90/Screenshot+2024-05-27+223855.png)
-   Select the users or groups you'd like to be able to access the resource.

### The Connector

We have to download the proxy connector and install it on the server we intend to access.

-   In the main blade, click **Connect** -> **Connectors**
-   On the **Private Network connectors** page, click **Download connector service** ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/2463949d-97dd-49c6-80c8-b96647fd47b4/Screenshot+2024-05-27+220648.png)
-   On the **Private Network Connector Download** page, click the **Accept terms & Download** button.

> The connector needs to be on your Windows Server. If you cannot add the setup.exe, then repeat the above steps to download the connector _\> directly_\> to the Windows Server.

-   Double click the _MicrosoftEntraPrivateNetworkConnectorInstaller.exe_ file to begin the installation. ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/2e86028c-13dc-4eb7-8094-4ff9aa5485fb/Screenshot+2024-05-27+221201.png)
-   Agree to the terms and click **Install** ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/4ed39b92-3966-4c36-9ec7-0bec43f29e84/Screenshot+2024-05-27+221444.png)
-   You'll then be prompted to authenticate with your Entra ID ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/ca88b183-1ef4-45f3-883f-c89b334e15d3/Screenshot+2024-05-27+221559.png)

Once the installation is complete, you will see the active connector on the **Private Network connectors** page. ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/c750821a-ab3e-4aea-b8c7-4abb37e17cf2/Screenshot+2024-05-27+221652.png)

### Deploy the client

Windows PCs will need the Global Secure Access Client installed on them in order to access the proxy service. We will deploy this with Intune.

-   Navigate to **Connect** -> **Client download** and click on the **Download Client** button for Windows 10/11 ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/90b45ae7-af39-4d99-8432-4048230a0b6c/Screenshot+2024-05-27+221758.png)
-   Use the [Microsoft-Win32-Content-Prep-Tool](https://github.com/Microsoft/Microsoft-Win32-Content-Prep-Tool) to create an **.intunewin** file ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/6a2ad500-35bd-4ef4-aab2-4dad7a996add/Screenshot+2024-05-27+222505.png)
-   Head over to **intune.microsoft.com** and navigate to **Apps** -> **Windows** -> **+Add**
-   In the **App type** drop-down menu, select **Windows app (Win32)** ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/d975da59-960d-4011-ba4e-de90f3f12ffc/Screenshot+2024-05-27+222658.png)
-   Click **Select app package file** and navigate to the **GlobalSecureAccess.intunewin** file we just created.
-   Fill out the _Name_, _Description_, and _Publisher_ fields.
-   Configure the remaining options as follows:
    -   **Program**
        -   Install command: _GlobalSecureAccessClient.exe /install /quiet_
        -   Uninstall command: _GlobalSecureAccessClient.exe /uninstall /quiet_
        -   Install behavior: _System_ ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/46a495fe-f8a2-4a03-8a35-fffbbc4f594c/Screenshot+2024-05-27+223006.png)
    -   **Requirements**
        -   Operating system architecture: _64-bit_
        -   Minimum operating system: _Windows 10 22H2_
    -   **Detection rules**
        -   Rule type: _File_
        -   Path: _C:\\Program Files\\Global Secure Access Client_
        -   File or folder: _GlobalSecureAccessClientChecker.exe_
        -   Detection method: _File or folder exists_
        -   Associated with a 32-bit app on 64-bit clients: _No_ ![](https://getrubixsitecms.blob.core.windows.net/public-assets/https://images.squarespace-cdn.com/content/5dd365a31aa1fd743bc30b8e/4497c3fc-59c0-4d55-8f4e-bf4602ebe82e/Screenshot+2024-05-27+223257.png)
-   For **Assignments**, I have chosen to make the app available to all users to download from the Company Portal.

Deployed!
---------

There we go; we now have the **Global Secure Access Client** app deployed to our PCs with Intune, ready to authenticate to our on-premises file share via the proxy connector.

In part 2, we'll go over the user experience and some advanced security options to further protect our app.
