---
author: steve@getrubix.com
date: Thu, 26 Mar 2020 14:56:49 +0000
description: '"When setting up an Intune/Autopilot environment, the rubric I cannot
  stress enough is “watch out for the little things”.&nbsp; You’d be genuinely surprised
  at how often one un-flipped switch or policy typo can sink the entire Windows 10
  deployment.&nbsp; Today I want to go over one of the"'
slug: automatic-enrollment-dont-be-afraid
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/automatic-enrollment-dont-be-afraid_thumbnail.jpg
title: Automatic Enrollment Dont Be Afraid
---

When setting up an Intune/Autopilot environment, the rubric I cannot stress enough is “watch out for the little things”.  You’d be genuinely surprised at how often one un-flipped switch or policy typo can sink the entire Windows 10 deployment.  Today I want to go over one of the most commonly overlooked aspects of the Modern Endpoint Manager (MEM)\*; automatic enrollment.

\*_Formally known as Intune_

To start off, let’s make sure we’re all on the same page.  Log in to [https://devicemanagement.microsoft.com](https://devicemanagement.microsoft.com) and navigate to **Devices -> Enroll devices -> Windows enrollment** and select “Automatic Enrollment”.

![Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1585233385215-MBQV2IPRYPUSEIUFHL1K/Microsoft+Endpoint+Manager+admin+center.png)

Here you’ll see the setting for **MDM user scope**.  I cannot recommend this enough; you should set this to “All”.  Usually when I bring this up, the info-sec folks get very nervous, tempers run high, and sometimes fights break out.  Well, maybe I’m exaggerating.  This stems from a fear that the enrollment gate is now wide open, with every device that ever breathed the same air as Azure getting enrolled without any control.  This is not the case.

Why ‘some’ is not enough.
-------------------------

![Discard.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1585233458246-5WS75USRDMQM48926PG2/Discard.png)

The problem with the “Some” option under **MDM user scope** isn’t the _some_ part at all.  It’s actually the _user_ part.  If you were gearing up for a straight user driven Autopilot enrollment, this wouldn’t be an issue.  But consider the options now available for Autopilot:

-   White glove
    
-   Hybrid join
    
-   Self deploy
    
-   White glove / hybrid join (stay away)
    

In all of these cases, the Windows 10 device being deployed presents itself to MEM before a user is in the picture.  No matter what groups are assigned under the **MDM user scope**,  a device by itself simply cannot get in.

![Enrollment scope.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1585233554920-77OP0VJSGLODOVAGE8BZ/Enrollment+scope.png)

So what we need is “All” as our setting for **MDM user scope**.  It makes more sense to think of it as ‘unrestricted’ or ‘open’.  It’s not that we’re opening it up to all of the users, but more like opening it up to everything, user or not.

![X Discard.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1585233598241-OFM3UL48CYXGA11RAMJU/X+Discard.png)

Now when the device is presented to MEM from Autopilot, it doesn’t matter that there is not a user object along for the ride.  There should be no restriction at all, thus allowing us to enroll in all the above mentioned scenarios that do not have a user at this stage.

![All users.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1585233634784-G444GCEAF8XD4TAFR8A8/All+users.png)

But now I’m wide open and scared!
---------------------------------

Naturally, it’s not enough that we’ve enabled White glove and Hybrid join deployments.  Now we have to deal with the fear.  And like most fears, this one comes from the unknown.  

There are two scenarios that are usually voiced as concerns with **MDM user scope** set to “All”- how to prevent unwanted mobile devices from enrolling and what happens to existing Azure registered PCs?

Oh, that’s what that’s for.
---------------------------

Turns out, the way to restrict unwanted enrollments is via the **Enrollment restrictions** setting.  In the MEM console, navigate to **Devices -> Enroll devices -> Enrollment restrictions**.  Here you can either edit the default policy or create a new one. 

![Microsoft Endpoint Manager admin center.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1585233722976-2XUSLTUQ1XKDZS1B7HXV/Microsoft+Endpoint+Manager+admin+center.png)

You have the option to restrict all platforms but Windows, or simply block devices that are not corporate owned (this can be determined via Autopilot, Apple Business Manager, etc).  In my tenant, I’ve gone ahead and blocked everything but corporate issued devices from enrolling.  

![Edit restriction.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1585233768330-ZLT4RKCFDVT1Z9XPWTAD/Edit+restriction.png)

So based on this, only my Autopilot registered PCs can enroll.  

But what about my existing machines?
------------------------------------

This one is a bit more confusing because honestly, the default behavior is not clearly documented.  If you have Windows 10 PCs that are Azure registered or Hybrid Joined, what’s to keep them from automatically enrolling?

Well as it turns out, they don’t just enroll themselves.  In order for this to happen, you need to use good, old-fashioned group policy.

![Auto MDM Enrollment with AAD Token.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1585233880452-87QZTLG5VGKAP04SB8OT/Auto+MDM+Enrollment+with+AAD+Token.png)

The **Auto MDM Enrollment with AAD Token** policy will govern which devices registered to Azure can be enrolled.  Simply enable this and link it to a specific OU.

![Group Policy Management.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1585233991868-5F4VK25FNNTJSI6UFOLX/Group+Policy+Management.png)

Existing devices now need to be moved to this OU in order to enroll in MEM.

Your call…
----------

Ultimately, this is your decision, and I do comprehend the gravity of enabling such a binary toggle.  It would be so much easier if Microsoft would allow us to set enrollment based on device group in addition to users.

But I think with the understanding of how it works and the options available, you can feel a bit calmer about enabling automatic enrollment and hopefully, have a successful Autopilot deployment.
