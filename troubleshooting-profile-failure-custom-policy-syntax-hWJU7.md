---
author: GuestUser
date: Sat, 30 Jan 2021 16:03:46 +0000
description: '"When deploying a custom policy, it is important to ensure the OMA-URI
  and Value fields are typed accurately. Refer to the Policy CSP document for correctly
  formatting OMA-URI entries: https://docs.microsoft.com/en-us/windows/client-management/mdm/policy-configuration-service-providerAlso,
  when entering the value of the policy, use the document above to ensure you are
  configuring a supported"'
slug: troubleshooting-profile-failure-custom-policy-syntax-hWJU7
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/troubleshooting-profile-failure-custom-policy-syntax-hWJU7_thumbnail.jpg
title: Troubleshooting Profile Failure Custom Policy Syntax
---

When deploying a custom policy, it is important to ensure the **OMA-URI** and **Value** fields are typed accurately. Refer to the Policy CSP document for correctly formatting OMA-URI entries: [https://docs.microsoft.com/en-us/windows/client-management/mdm/policy-configuration-service-provider](https://docs.microsoft.com/en-us/windows/client-management/mdm/policy-configuration-service-provider)

Also, when entering the value of the policy, use the document above to ensure you are configuring a supported value for the policy. The document will help confirm whether the value type is a string, integer, Boolean, or different data type. The document also provides examples of the supported values and correct syntax, which can very between different policy types.

For String values, copying and pasting can even be the culprit at times… you may have copied unwanted symbols that didn’t translate in the browser correctly. pay special attention to quotation marks, as they should be straight instead of curly:

“This will break policy”

"This will work in policy"

For custom policies that require json, xml, or character data, ensure that you are following the guidelines and examples from Microsoft. If configuring third-party ADMX backed policies, the company may have their own documentation on how to format Intune-specific policy. One example of a Microsoft policy is managed favorites in Edge. When browsing the policy in Administrative Templates, they provide a sample json value which the policy requires:

![Picture1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1612022354246-80AEQP0LE7ZU1RQFMZ2O/Picture1.png)

As long as you keep the formatting exactly the same, this policy should work with your custom bookmarks. Interestingly though, if you configure the same policy for Firefox or Chrome, they require slightly different formatting. Here’s the **Firefox** variant according to their policy documentation ([https://github.com/mozilla/policy-templates/blob/master/README.md](https://github.com/mozilla/policy-templates/blob/master/README.md)):

**Value (string):**

```
<enabled/>
<data id="JSON" value='
[
  {
    "toplevel_name": "My managed bookmarks folder"
  },
  {
    "url": "example.com",
    "name": "Example"
  },
  {
    "name": "Mozilla links",
    "children": [
      {
        "url": "https://mozilla.org",
        "name": "Mozilla.org"
      },
      {
        "url": "https://support.mozilla.org/",
        "name": "SUMO"
      }
    ]
  }
]'/>
```

Aside from the <enabled/> and <data id/>, the formatting is nearly identical to the Microsoft Edge version of the policy. Note that the json value is all contained within single quotes due to the inner components having double quotes.

For **Google Chrome**, there is some additional pickiness where you need to remove the spacing in the json (you can use [https://removelinebreaks.net](https://removelinebreaks.net)). It’s strange because Edge and Firefox base the bookmarks policy off of Chrome, and you can technically remove the spaces for any of the browsers’ policies, but Chrome will not work until the spacing is formatted as such:

 <data id='ManagedBookmarks' value='\[{"toplevel\_name":"My managed bookmarks folder"},{"url":"google.com","name":"Google"},{"url":"youtube.com","name":"Youtube"},{"name":"Chrome links","children":\[{"url":"chromium.org","name":"Chromium"},{"url":"dev.chromium.org","name":"Chromium Developers"}\]}\]'/>

  
While Google Chrome does not document all of their policies in Intune format, they do have examples of commonly used policies on this link: [https://support.google.com/chrome/a/answer/9102677](https://support.google.com/chrome/a/answer/9102677)
