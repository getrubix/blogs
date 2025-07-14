---
title: ADMX in Intune- a year or two later Part I
slug: admx-in-intune-a-year-or-two-later-part-i
date: "Thu, 24 Dec 2020 16:01:47 +0000"
author: steve@getrubix.com
description: 'When I started writing this piece, the plan was to title it "ADMX: a year later". I was going to reference my original write up on ADMX policy in Intune and highlight what has changed since then. Well, as soon as I started, I realized I could not find the original.'
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/admx-in-intune-a-year-or-two-later-part-i_thumbnail.jpg
---

When I started writing this piece, the plan was to title it "ADMX: a year later".  I was going to reference my original write up on ADMX policy in Intune and highlight what has changed since then.  Well, as soon as I started, I realized I couldn't find the original.  Then it hit me; I wrote that back in 2018, not 2019.  In fact, getrubix.com wasn't even a thing at that point.  So I suppose instead of referencing something you've never read, here comes a two-parter.  In the first part, I'll walk through the current state of ADMX policy ingestion via Microsoft Intune.

## Ingestion
---

To configure third-party ADMX policiy with Intune, we need to create a custom profile.  To do this, log into the Intune console at [endpoint.microsoft.com](https://endpoint.microsoft.com) and select **Device -> Device configuration -> Profiles -> Create profile -> Windows 10 and later -> Custom**.  

We will then need the correct **OMA-URI**, **Data type** and **Value**.

### OMA-URI

Custom policy can be set in Intune using the CSP guide found at [https://docs.microsoft.com/en-us/windows/client-management/mdm/policy-configuration-service-provider](https://docs.microsoft.com/en-us/windows/client-management/mdm/policy-configuration-service-provider).  This, in general, is the best place to explore all Windows 10 policies that can be applied with Intune.  For ADMX template ingestion, we need a specific OMA-URI:

```
./Device/Vendor/MSFT/Policy/ConfigOperations/ADMXInstall/{App Name}/{Setting Type}/{ADMX name}
```

Everything up to _ADMXInstall_ is standard for all ingestions.  The variables that follow get their values based on the ADMX file itself, in this case, _Chrome.ADMX_.  This is how we set these.

![Screen Shot 2020-12-24 at 10.39.38 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1608824796830-XAP2OIM4JEI6E7HEO89N/Screen+Shot+2020-12-24+at+10.39.38+AM.png)

**{App Name} = "Chrome" (prefix)**

**{Setting Type} =** Policy (always).  For almost all ADMX templates, this setting type will be "policy"

\*IMPORTANT NOTE: when setting "Policy" for the ingestion schema, pay attention to the case sensativity.  It does not matter whether you choose "policy" or "Policy", but once it is set here, you must be consistent with all policies that follow.  So if you choose to use "policy", all future policies set against this ADMX must read "policy".  I recommend staying uppercase.

**{ADMX name} =** chromeADMX (name of file, but can technically be anything).  Because we cannot insert a period in the OMA-URI schema, we can turn 'chrome.admx' into chromeADMX.  My recomendation is to use camel case here.

Based on these variable explanations, our ingestion OMA-URI will read as follows.

```
./Device/Vendor/MSFT/Policy/ConfigOperations/ADMXInstall/Chrome/Policy/chromeADMX
```

### Data Type

The data type will be "String" \*

### Value

Inside the value field, paste the ENTIRE CONTENTS of the 'Chrome.admx' file.  You can now click 'OK' and create the policy.

> \*Do not choose 'String (XML file)' and upload.  This does not work

## Setting Policy
---

Once the ADMX template has been ingested through Intune, we can set individual policy against it.  This will involve some searching through the 'Chrome.admx' file so keep it handy for reference.  In this example, we will set a policy to disable Chrome's Incognito Mode.  Like before, Intune requires three components: the **OMA-URI, Data type** and **value**.

### OMA-URI

The CSP schema for configuring an ADMX-backed policy is as follows:

```
./Scope/Vendor/MSFT/Policy/Config/AppName~Policy~ParentCategory/PolicyName
```

So let’s stick with Chrome as an example.  In this case, we are going to block the ability for private browsing.  Here is the policy in the original Chrome ADMX that was ingested:

![Screen Shot 2020-12-24 at 10.51.49 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1608825265334-2WQFWLNWMK3K2D4N1L6G/Screen+Shot+2020-12-24+at+10.51.49+AM.png)

So now, we can match up the variables we need with the correct values.  These are highlighted above.

**Scope:** Device (_Policy Class=”both”; we will always choose ‘Device’ when “both” is an option_)

**Parent Category:** “googleChrome”

**Policy Name:** “IncognitoModeAvailability” (_name=”value”_)

So our OMA-URI to set the policy for Chrome’s incognito mode will read as follows:

```
./Device/Vendor/MSFT/Policy/Config/Chrome~Policy~googlechrome/IncognitoModeAvailability
```

### Data Type

Data types for ADMX policies are always “string”

### Value

With the OMA-URI set for configuring Chrome’s incognito mode, we can now set the value.  An ADMX policy value can have either 1 or 2 components.  First is a simple “enabled” or “disabled” value:

```
<enabled/>
```

```
<disabled/>
```

If the policy has only an on or off option, we would stop there.  However, sometimes there is another piece; the “data ID” and “value”.  Take a look:

```
<enabled/>
```

```
<data id=”somevalue” value=”X” />
```

Let’s look back on the Chrome ADMX for this policy.  The information we need will always be inside the <elements> brackets:

![Screen Shot 2020-12-24 at 10.52.54 AM.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1608825362697-EQM1GPE0Q4R1GWGLSDSU/Screen+Shot+2020-12-24+at+10.52.54+AM.png)

So because there is more than a simple on or off to this policy, we know we have the two parts.  Let’s start by enabling it.

```
<enabled/>
```

Do you see the “enum id=” piece?  That will act as our data ID.  Essentially, whenever a policy has a “**_ABC_** ID”, we will assume it is the data ID.  In this case, it will read:

```
<enabled/>
```

```
<data id=”IncognitoModeAvailability”
```

 We still need a value.  Above, you can see this policy has three different values; 0, 1 and 2.  Which one do we use?  Well at the beginning of each value, there is an item string describing the policy behavior. 0 = “IncognitoModeAvailabity\_Enabled”, 1 = “IncognitoModeAvailability\_Disabled” and 2 = “IncognitoModeAvailability\_Forced”.  

We want to disable this, so we will use “1” as our value.  So now the completed string should read as:

```
<enabled/>
```

```
<data id=”IncognitoModeAvailability” value=”1” />
```

![10.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1608825482474-BYFFTTXHFSL178UDU1NJ/10.png)

Easy, right?
