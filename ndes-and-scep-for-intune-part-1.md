---
author: steve@getrubix.com
categories:
  - intune
  - security
  - automation
  - azure
date: Mon, 10 May 2021 20:52:48 +0000
description: >
  Long time, no talk. But it’s because I’ve been busy. And usually when I’m this busy it means I’ve got a lot to talk about. During three separate Endpoint Manager implementations, I’ve recently had to go outside my comfort zone and help folks troubleshoot Intune SCEP certificate profiles.
slug: ndes-and-scep-for-intune-part-1
tags:
  - endpoint manager
  - azure
  - active directory
  - security
  - intune
  - flow
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/ndes-and-scep-for-intune-part-1_thumbnail.jpg
title: NDES and SCEP for Intune Part 1
---

Long time, no talk. But it’s because I’ve been busy. And usually when I’m this busy it means I’ve got a lot to talk about.

During three separate Endpoint Manager implementations, I’ve recently had to go outside my comfort zone and help folks troubleshoot Intune SCEP certificate profiles. That led to poking around the SCEP connector itself. Once I started looking there, it wasn’t long before I pumped the breaks, took a deep breath, and figured out how to build the whole thing from scratch.

So, sit back and relax while I take you through the entire setup process of an Intune certificate connector on a fresh, new NDES server.

## Mini Series

There’s a lot of things that need to happen in order to get this working properly. Anyone who tells you it’s ‘painless’ or ‘no big deal’ is a heartless liar. It’s confusing, frustrating and worst of all, there’s little documentation of the entire process in its entirety.

I found very good pieces written by various tech resources detailing specific parts, often one blog had a piece missing from the other, all becoming pieces of the larger solution.

So, what I’ll do here is break this into several parts of a whole series, each piece detailing their own part of the process. This way it can stay manageable, but all reside in the same place.

## Workflow

The high-level breakdown is as follows:

- NDES is a Windows Server joined to your Active Directory. **Do not use a domain controller for this.**
- NDES contains IIS role, which will handle incoming web requests from Intune asking for certs.
- Azure application proxy is used to provide an external URL that points to the internal URL of the NDES.
- Intune certificate connector is installed on NDES.
- Intune SCEP profile makes request through Intune Certificate connector for cert. NDES asks for cert template from issuing CA and deploys through Intune.

## Why Do I Need This?

The Intune certificate connector lets you deploy certificates to devices that you would traditionally deploy to a domain joined PC via group policy.

But we’re not domain joined anymore, are we now? No, we’re not. So we need a way to get those same certs from the Domain CA that are used for client authentication for VPN, MFA, and other fun things.

Alright, here we go. For clarity, each section will have a location code so you know exactly where we’re performing each step.

**Codes:**

- CA = Certificate Authority  
- NDES = Network Device Enrollment Service (server we’re building)  
- Intune = Microsoft Endpoint Manager (https://endpoint.microsoft.com)  
- AD = anywhere in your Active Directory  

## Part 1 – The Service Account, Certificate Templates, and NDES Role

### Make an NDES Account and Server (AD)

In your on-premises Active Directory, create a new user that we will use as a service account for our NDES activities.

![Creating an NDES service account](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679404895-51WR7LL9HJ3INX3UMCKC/1.png)

For the server, just spin up a fresh Windows Server 2016 or later physical or virtual machine and join it to your domain. **Do not promote it to a domain controller.**

### Certificate Templates (CA)

We will make two certificate templates. First will be the Web Server template used for NDES and Intune connector authentication to the CA.

Next is the SCEP template for client authentication—this will be the certificate that gets issued to Intune devices via connector.

Log into your CA, open the Certification Authority. Expand the CA and right-click Certificate Templates. Click **Manage**.

![Open Certificate Templates](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679578521-PVSR27LPCQF6BDAZI2JK/2.png)

The Certificate Templates Console opens. Right-click on **Web Server** and select **Duplicate Template**.

![Duplicate Web Server Template](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679599831-FYOAFIGZTDFTG69UNKM6/3.png)

In the **Extensions** tab, edit Application Policies to contain:

- Server Authentication  
- Client Authentication  

![Application Policies](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679644650-E9LVHMA1CU2ZZXRGY941/4.png)

In the **Subject Names** tab, ensure that **Supply in the request** is selected.

![Subject Names Tab](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679665286-TB5ZEBZ91C7O3TDDKD1I/Picture5.png)

In the **Security** tab, add the name of the NDES server and grant:

- Read  
- Enroll  
- Autoenroll  

![Security Permissions](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679731146-4V15VCR5O0415LRUF280/Picture6.png)

Name it clearly in the **General** tab (e.g., “NDES”). Click **Apply** and **OK**.

Use the same process to make another certificate. This one will be the client authentication template for Intune. Use these settings:

- Duplicate from “User” template  
- Application Policies → Add **Client Authentication**  
- Security → Add NDES user with **Read, Enroll, Write, Autoenroll**  
- Subject Name → Select **Supply in the request**  
- Click **Apply** and **OK**  

Back in the Certification Authority console, right-click **Certificate Templates** and select:

**New → Certificate Template to Issue**

![Issue Certificate Templates](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679757930-N6I4J7017EXQ6C4M44G0/Picture7.png)

Select both templates you just created and click **OK**.

### NDES Role (NDES)

Log into the NDES server. Launch **Server Manager** and click:

**Manage → Add Roles and Features**

Add the following roles:

- **Active Directory Certificate Services**  
- **Web Server (IIS)**  

![Add Roles](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679812712-IU1UD3FUFK5XYE1ZXDZH/Picture8.png)

Under Web Server role, select all required features:

![Web Server Features](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679841919-G4JAM8L0MVH7EMDZRRI8/Picture9.png)

In **Features**, check these options:

![Additional Features](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679867119-MKM39ZH8I890033RXVB0/Picture10.png)

For Active Directory Certificate Services role services, uncheck all but:

- **Network Device Enrollment Service**

![NDES Role Selection](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620679891567-4CM2KRGHQ2DDYQX1U4XO/Picture11.png)

When prompted:

- For Service Account → use the NDES user  
- For Certificate Authority → choose **Computer name**, enter the FQDN of your CA  

Click **Next** until completed. Restart the NDES server.

---

Congratulations — you’ve completed Part 1. Better get some rest before Part 2.
