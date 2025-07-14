---
author: steve@getrubix.com
date: Thu, 16 Jan 2025 15:55:07 +0000
description: '"My journey in the IT world began with network security in the Marines,
  where I gained all sorts of invaluable experiences. After some time, I eventually
  shifted my focus to endpoints. When I heard about Microsoft''s release of their
  Security Service Edge solution - which combined both of"'
slug: exploring-microsoft-entra-private-access
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/exploring-microsoft-entra-private-access_thumbnail.jpg
title: Exploring Microsoft Entra Private Access
---

My journey in the IT world began with network security in the Marines, where I gained all sorts of invaluable experiences. After some time, I eventually shifted my focus to endpoints. When I heard about Microsoft's release of their Security Service Edge solution - which combined both of my interests – I was very curious to learn more.

In July 2024, the Microsoft Entra Suite was introduced. I was at Microsoft when they announced this with Steve as part of an NDA event, and they gave us Entra Suite shirts and all sorts of nice swag. We couldn’t wear our shirts until July, but luckily mine still fit me then.

The Entra Suite packs a ton of great features - for this blog, I want to focus on one standout: **Private Access**. This feature makes it easy to securely connect to private applications and on-premises resources, all while staying true to **Zero Trust Principles**. It’s a powerful solution that blends security with usability, and I can’t wait to dive into it further.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7a6625d9-0c94-4db4-8a5a-39378262a375/dgblog.png)

### **  
Setting Up Private Access**

Getting started with **Private Access** is pretty straightforward. The first step is enabling the **Traffic Forwarding profile** in Microsoft Entra. Think of traffic forwarding as the traffic cop of your network—it ensures that data from endpoints is securely and efficiently routed through the Entra service. It’s all part of the Zero Trust network access (ZTNA) framework, meaning every bit of traffic is authenticated, encrypted, and monitored before it reaches your corporate resources.

To set this up, just head to **Global Secure Access > Connect > Traffic Forwarding** and enable the **Private Access Profile**.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/3ba42ace-7ad0-4646-b9a0-12efc444e3cb/dgblog2.png)

The next step is setting up the **Private Network Connectors** in Entra. Don’t worry, it’s not as complicated as it sounds. You’ll just need an account with at least the **Application Administrator** role in Entra to get started.

Here’s how you do it:  
  
Go to **Global Secure Access > Connect > Connector** and download the **Connector Service**. Once you’ve got it, install the service on a member server. Microsoft has some detailed documentation on setting these up, so if you’re curious or need a deeper dive, you can check that out [here](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-configure-connectors).

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ae626378-7d0f-4615-a013-a7e1532cc5bd/dgblog3.png)

The installation of the connector is straightforward. After launching the executable, click the install button. You will be prompted to enter your Entra credentials, which need to be at least Application Administrator in Entra. After entering your credentials, you will be prompted to restart the server.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/260c0dbe-7b76-4611-8133-bb1ec384aa07/dgblog4.png)

You will be prompted to enter your Entra credentials. These credentials need to be at least Application Administrator in Entra.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/8e29dfe4-e86b-4d71-b41b-4f4dac11bc65/dgblog5.png)

You can organize the connectors into connector groups. By grouping multiple connectors, organizations can achieve high availability and load balancing, ensuring consistent performance and fault tolerance.

### **Creating an Application Segment**

With the connector up and running, it’s time to set up an **application segment**. Think of these segments as neat little groups of resources—whether it’s URLs, domains, IP addresses, or ports—that let you fine-tune your access policies. They’re a great way to enforce **Zero Trust principles** while keeping things organized and secure.

Here’s how to do it:

Head to **Global Secure Access > Applications > Enterprise Applications**. Click on **New Application**, give it a name, and choose the **connector group** that includes the connector we just installed. Then, select **Add Application Segment**, and you’re all set!

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ac098c67-aba9-4364-8147-588fa354892a/dgblog6.png)

Since this application will be allowing RDP to this server, I want to allow port 3889 for the FQDN name of the server.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/649a096a-76c5-438d-9ed8-e89d8e9f9b90/dgblog7.png)

Once we save the application and exit, we can see that the application now shows. You will need to allow access to your users and/or groups that need access to this application.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/edd9ebab-c31f-4173-b71d-f59e9e6abace/dgblog8.png)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e01b3caf-ae75-44fe-900e-eeb2a75200bc/dgblog9.png)

### **Controlling Access**

Now that we’ve got our application set up, it’s time to think about access control. Using Conditional Access, we can fine-tune who gets in and how. And don’t worry—we’ll dive into all the details in a future blog, along with installing the GSA client on a device for some hands-on testing. Stay tuned!
