---
author: steve@getrubix.com
date: Mon, 10 Feb 2025 13:03:56 +0000
description: '"Simplifying Secure Access with Microsoft Entra ID GovernanceManaging
  access to critical resources can be a nightmare like trying to keep snacks away
  from a hungry toddler. You want to lock things down tight, but you also don’t want
  to deal with endless requests for “just five more minutes”"'
slug: now-you-see-it-now-you-dont-secure-access-with-entra-id-governance
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/now-you-see-it-now-you-dont-secure-access-with-entra-id-governance_thumbnail.jpg
title: Now You See It Now You Dont Secure Access with Entra ID Governance
---

**Simplifying Secure Access with Microsoft Entra ID Governance**

Managing access to critical resources can be a nightmare like trying to keep snacks away from a hungry toddler. You want to lock things down tight, but you also don’t want to deal with endless requests for “just five more minutes” of access.

Enter [Microsoft Entra ID Governance](https://learn.microsoft.com/en-us/entra/id-governance/identity-governance-overview) and its **Access Packages** the superhero we didn’t know we needed. These magical bundles let organizations automate access requests while ensuring security doesn’t take a backseat. Think of them as an all-you-can-eat buffet, but instead of food, it’s access to apps, groups, and resources with strict portion control**.**

**But Wait, What About Just-in-Time (JIT) Access?**

In a previous blog, I covered [Entra Private Access](https://www.getrubix.com/blog/exploring-microsoft-entra-private-access), which lets us assign access to users or groups for specific applications. But what if we don’t want users to have permanent access? What if we need Just-in-Time (JIT) access for high-value resources, like domain controllers?

Since my [Microsoft Entra Suite](https://www.microsoft.com/en-us/security/business/microsoft-entra) license includes both Entra Private Access and ID Governance, I decided to combine them. This setup lets users request access on-demand (JIT) and securely connect to their on-premises resources only when needed—kind of like a bouncer checking IDs at an exclusive club, except the club is a domain controller, and the bouncer is an access policy.

Let’s break it down step by step.

 **Setting Up Access Packages for Secure RDP Access**

To illustrate how this works, I’ll walk you through configuring an Access Package for RDP access to domain controllers using Entra Private Access.

**Step 1: Create a Catalog**

First, navigate to **Identity Governance > Catalogs**. A catalog is essentially a container that organizes Access Packages along with related resources like groups, applications, and SharePoint sites. It allows administrators to delegate access management while ensuring only authorized users can request or assign access.

Instead of adding resources to the default General Catalog, I created a new one called Rubix Administrators. This catalog will manage RDP access to domain controllers and other critical resources (I’ll cover these additional resources in a future post).

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/03ef5c33-3b34-45bc-81de-22931fe02af2/blog1.png)

**Step 2: Add Resources**

Once the catalog is created, it’s time to add the resources that the **Rubix Administrators** group will need. For this setup, I added:

-   My Private Access applications for both my Domain Controller and MECM server.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/62464206-4f46-447a-86da-13211ae20e05/blog2.png)

**Step 3: Create an Access Package**

Now, head to **ID Governance > Access Packages** to create the actual package.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/b5320456-c923-4810-9a16-21471329b4af/blog3.png)

**Name the Access Package** and select the **Rubix Administrators** catalog.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/510ea9aa-9b7b-4eaf-8dba-41df348a211d/blog4.png)

**Choose Resources**: While the catalog may contain multiple resources, for this example, I’m only including my **Domain Controller RDP application**.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/81c84678-8207-4910-8bdf-486265e95564/blog5.png)

**Configure Requests**: Here, I add my **Rubix Administrators group** and define approval requirements.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/dc5bb79a-607c-4edd-be9f-28ba266f1d9a/blog6.png)

**Request Form:** If you want to make users sweat a little, you can ask them additional questions before granting access. (Optional, but great for passive-aggressively reminding people that security is serious business.)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/90215f22-8763-4667-8db7-02d48dbf1bce/blog7.png)

**Lifecycle Policies**: I set this Access Package to expire **after two hours** since it grants **RDP access to a domain controller**.

**Access Reviews:** Not covering them today, but they’re like those “are you still watching?” prompts on Netflix except for security.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/76589ea2-8475-497c-9497-0facb788710d/blog8.png)

Custom extensions are available as an optional tab. Currently, I do not have any Logic Apps in my test environment, but the [Microsoft documentation](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-logic-apps-integration) provides detailed coverage on this topic.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/2e3bc2ad-e486-4769-bbc1-7409a7048ea7/blog9.png)

**Step 5: Testing Access Requests**

Now, let’s test it out!

1.  A user attempts to **RDP into the domain controller** but gets an error expected, since they don’t yet have access.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e2a7fc07-f3bc-4dc8-95cd-ae60a2187262/blog10.png)

2\. The user navigates to [**https://myaccess.microsoft.com/**](https://myaccess.microsoft.com/) and selects the relevant **Access Package**.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/954cbdfe-576a-48cd-912a-2477e6b1825d/blog11.png)

3\. If a **Business Justification** was required during setup, they fill it in.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/27cbe555-b2d8-43a2-bcb3-2d1cffc5a084/blog12.png)

4\. Once the request is **approved**, the user is automatically added to the required application.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/f267bf32-a737-4f14-8c30-aa2b351f0369/blog13.png)

5\. The user can now **RDP into the domain controller** using **Entra Private Access**.

And the best part? Since we configured a **two-hour expiration**, access is **automatically revoked** once time’s up no manual cleanup needed.

**Wrapping Up**

By combining **Entra Private Access** with **Access Packages**, organizations can implement **Just-in-Time access** without compromising security. Users get the access they need **only when they need it**, and IT teams stay in control with automated approvals and expiration policies.

Stay tuned, I’ll be diving deeper into **Access Reviews** in a future post. Until then, happy securing!
