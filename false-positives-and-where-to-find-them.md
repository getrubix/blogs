---
author: Guest
date: Fri, 14 Feb 2025 12:41:26 +0000
description: '"Purview can generate a significant number of false positives from its
  built-in Sensitive Information Types. Part of my job when running datasecurity projects
  is to help resolve these and keep them from popping up again in the future. Some
  of this may be common sense, and some of"'
slug: false-positives-and-where-to-find-them
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/false-positives-and-where-to-find-them_thumbnail.jpg
title: False Positives and Where to Find Them
---

Purview can generate a significant number of false positives from its built-in Sensitive Information Types. Part of my job when running datasecurity projects is to help resolve these and keep them from popping up again in the future. Some of this may be common sense, and some of it may be new to you...either way, let's dive in:

## Analytics and Initial Assessment

One of the more recent options for reporting on DLP is Data Loss Prevention analytics. I highly, highly recommend you enable this if you haven't already:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/66f1f4a0-531c-444f-8072-7005e6879690/blog1.jpg)

Enabling Data Loss Prevention Analytics in Microsoft Purview

-   If you already have DLP policies configured in your environment, you should review specific DLP rule match hits in the Activity Explorer.
    

-   Data Loss Prevention Analytics can take a few days to start showing data, so this is a good intermediate step to start getting a handle on your sensitive data.
    

## Confidence-Level Triage

Each SIT has multiple confidence levels (low, medium, and high) that are essentially different levels of accuracy. Filtering your reports by Confidence Level gives you a more actionable idea of what you're dealing with.

-   It's worth noting that if you have, say, 1000 hits on Low Confidence Social Security Numbers, you might not necessarily have that many Social Security Numbers floating around.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/d224a982-6afb-4a99-b245-751d5f3e5a67/blog2.jpg)

Spot check a few hits under each confidence level to get an idea of what you're dealing with.

## Customizing SITs and Managing Exceptions

If you're dealing with Low Confidence Social Security Number hits that are actually unformatted 9 digit numbers that, for example, represent your customer account numbers, you have a few options:

-   Copy the built-in Social Security Number SIT and build exceptions for your customer account numbers, or:
    

-   Use the built-in Social Security Number SIT in your DLP policies, and build exceptions into the DLP policies themselves.
    

_Of course, one of these scales better than the other, but it's worth knowing all of your options_.

## DLP Policy and Rule Structure

You probably won't ever get your SITs 100% accurate (_if you do, let me know. I'd love to build a case study_). One method that I use to account for this is to build (3) rules per policy:

_Low Confidence Rule_

-   This rule will trigger on low confidence SSN hits and generate a policy tip with "non-aggressive" text, something to the effect of:
    

**"Potentially sensitive PII data may exist in this email. If this is an error, you can ignore this and continue."**

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/fd117c7c-5720-4048-ad67-5ee7b0b92a4b/blog3.jpg)

_Medium Confidence Rule_

-   This rule will trigger on medium confidence SSN hits and generate a policy tip formatted something like:
    

**"Sensitive PII data may exist in this email. Please consider a more secure format when working with PII. If this is an error, you can provide a justification and override the policy."**

-   I would then setup a "block only people outside your organization" action with an option for the user to override it with a business justification.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7fe5c323-61ee-4f5c-9f81-7c7e29b15840/blog4.jpg)

_High Confidence Rule_

-   This rule will trigger on high confidence SSN hits and generate a policy tip similar to:
    

**"Sensitive PII data has been detected in this email. This message will be blocked. If this is an error, please contact IT Security."**

-   Then setup an action of "block only people outside your organization" with **no** option configured to override.
    

-   Optionally, you could also configure an administrator alert for this.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/6b96eeb4-3b24-42bf-b94d-2ee60c23105c/blog5.jpg)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/d3eee83d-9c11-432a-8f8d-dfcc6675fdbb/blog6.jpg)

Maybe I'll come up with some automation for this :)

What About?
---

### Exact Data Match (EDM)

If your organization has sensitive structured data (like customer accounts), consider using an EDM Classifier to further reduce false positives by matching data in a more scalable fashion.

-   This is especially great for financial services orgs where you're constantly adding and removing customers from your records.
    

### Advanced Conditions

You can also use additional DLP Rule conditions (e.g., character proximity, document property, etc.) to further refine your SIT detection.

-   Just be careful modifying the default character proximity to anything above 300. The larger the number, the more likely you are to encounter DLP engine timeouts.
    

-   Be _especially_ cautious when using the "Anywhere in the document" option. Honestly, I'd just never consider using it.
    

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/89a11200-778b-4daf-8478-f73bf0166495/blog7.jpg)

I know the checkbox is enticing, just...don't.

Other Notes
-----------

### Review User Overrides

If you allow user overrides at the medium confidence level, track how often these are actually used.

-   If you notice frequent overrides, this might mean there are false positives that need to be addressed or that your users need more training.
    

### Communicate!

As you update your SITs and DLP policies, make sure you keep your users informed so they understand new notifications and rule actions.

-   This ensures you reduce help desk calls, giving you more time for more important things (like meetings that could have been emails).
    

One More Thing...
-----------------

Most of the built-in SITs have an entity definition page that explains how they work. Here's one for US Social Security Number:

[U.S. social security number (SSN) entity definition | Microsoft Learn](https://learn.microsoft.com/en-us/purview/sit-defn-us-social-security-number)

If you're having issues with your Purview Data Security efforts, let's connect and see where I can help!
