---
author: steve@getrubix.com
date: Wed, 12 Feb 2025 11:28:14 +0000
description: '"One of the most common issues I''m tasked with resolving for my Purview
  customers is the structure of the DLP Rules within their DLP Policies. You build
  a rule based on common sense understanding and find that common sense isn''t always
  the right approach. It''s easy to see"'
slug: complex-dlp-rule-design-is-wellcomplex
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/complex-dlp-rule-design-is-wellcomplex_thumbnail.jpg
title: Complex DLP Rule Design Is WellComplex
---

One of the most common issues I'm tasked with resolving for my Purview customers is the structure of the DLP Rules within their DLP Policies. You build a rule based on common sense understanding and find that common sense isn't always the right approach.

It's easy to see the Boolean operator options in front of you and take them at face value. Looking at this rule logic, you would be forgiven for assuming that it would match on an email containing a Social Security number **and** an ITIN:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ff844dbe-0275-4387-932e-ac349db6b112/blog1.jpg)

Unfortunately, that's not how Purview is expecting that condition to be built:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/9284562b-6ce1-4990-b317-37c61ffe99ea/blog2.jpg)

What you _should_ do, is this:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/bd4ea657-6fe3-4904-a1c1-f88467713657/blog3.jpg)

use the "create group" option within the current "contain contains" condition

Now we can add the Sensitive Information Type (SIT) that we want to link:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/ac15ff3d-1e12-4c10-8bba-d82823d47db3/blog4.jpg)

You would think that these are functionally the same, but Purview doesn't evaluate the first one as a Boolean "and". If you've built your rule that way, and you're not seeing the expected hits in the Activity Explorer, modify it to use a nested group.

### ❓Ok, but what about...

One thing to note is that there are still scenarios where you should use a non-nested group. Here, you can see that nesting a "Content Contains" condition only leaves a few options for the second condition:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/678bd76b-0e37-4f2e-a4b3-d77ee181783d/blog5.jpg)

when nesting a "content contains" group, your only options are SIT, Sensitivity Labels, or Trainable Classifiers

But, adding a second condition gives you way more options (like, way-way more):

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/76f2ed39-fc95-4415-8d9f-031dcba294bc/blog6.jpg)

looks a lot like Exchange Online mail flow rules, huh? :)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/0c77458b-b881-4d8b-ae2c-2395409477f3/blog7.jpg)

building Boolean "and" conditions without nesting

### ➕Bonus Options

One or the other:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/06df4610-0154-487d-a8be-045a7e3539f5/blog8.jpg)

One or the other, _and_, one or the other:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/7e913585-7392-442e-bb31-7a51f3123b03/blog9.jpg)

this is getting out of hand, now there are two of them

One or the other, _and_, one or the other AND another:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/4efc4e08-5e3e-4da8-a542-56be7d6f2114/blog10.jpg)

and and, Brute?

Hopefully this somewhat simplifies the complex. If you're having issues with your Purview DLP policies, let's connect and see where I can help!
