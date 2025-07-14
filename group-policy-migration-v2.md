---
author: steve@getrubix.com
date: Thu, 31 Mar 2022 02:12:41 +0000
description: '"Wow! So first off, thanks for all of the extremely positive response
  to the last post. While it was understandably rough, I’m glad enough folks were
  able to put it to use. Today I’m excited to unveil the V2 of this PowerShell script,
  complete with"'
slug: group-policy-migration-v2
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/group-policy-migration-v2_thumbnail.jpg
title: Group Policy Migration V2
---

Wow! So first off, thanks for all of the extremely positive response to the [last post](https://www.getrubix.com/blog/group-policy-to-intune-migration). While it was understandably rough, I’m glad enough folks were able to put it to use. Today I’m excited to unveil the V2 of this PowerShell script, complete with way less steps than V1.

Before I start, I need to give a massive shout out to Logan Lautt, an Azure developer on my team, who helped me get this script where it needed to be. Hopefully, you’ll be hearing from Mr. Lautt soon enough in an upcoming post.

The new, new
------------

To get things started, go ahead and download the new version [here](https://github.com/groovemaster17/IntunePowershell/blob/main/gpoToCspMigration_V2.ps1). So what’s different?

### Modulation

With V1, there was a custom authentication method that was able to access the Microsoft Graph endpoints with an Azure application registration. I did receive a number of questions about why we do that. In this particular case, it was just the method I was using when creating V1.

The main benefit is that there are no modules required; it relies solely on web requests. Why is that a benefit? Often times we push PowerShell scripts to endpoints via Intune for various reasons. If we want them to authenticate to the graph, it’s best not to have to push required modules out to those endpoints. However, that is obviously a different use case. So we streamlined things a bit by leveraging the [Microsoft.Graph.Intune](https://www.powershellgallery.com/packages/Microsoft.Graph.Intune/6.1907.1.0) PowerShell module in V2.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/20df7112-c10f-4ca1-91ed-ea8812ab1045/2.png)

As you can see, we start by checking if the module is installed, and if not, installing it before importing. With the Microsoft.Graph.Intune module installed, we can simply call **Connect-MSGraph** to authenticate. You will need to sign-in with Intune Admin credentials (or higher) to use the script.

### Required Parameters

There are two variables that must be modified in order for the script to function. First is the **$csvPath** which points to the exported policy csv file from Intune. The other is **$policyName** which is the name we want the configuration profile to have when it posts to Intune. Instead of hard-coding those in the script, they are now required parameters to make it easier when running.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/09ec5d4b-7afd-415a-a710-158a7b2e2c15/1.png)

Why didn’t we use those to start? That is a fantastic question and one of the first that Logan asked me when working on this. If you asked yourself the same thing, you’re in good company (the answer is I’m lazy).

### Check check

Finally, there is no more requirement to modify the csv file before hand. If you used V1, then you probably noticed we had to change a few things in the exported policy analytics report:

-   Header titles
    
-   Unsupported settings
    
-   Duplicate settings
    

Now we are taking care of all those pieces in the script logic. We convert the properties of the policy with the header titles that are in place. Then, the setting list is checked so that duplicates do not make it to the conversion queue.

Finally, only the **MDM Supported** values that contain “Yes” are migrated. We breeze right past “Deprecated” and “No”.

Export and run
--------------

And that’s all there is to it. To convert a policy, just export the Group Policy Analytics report from Intune, and run the PowerShell script. Just like before, your policy will be converted to an Intune device configuration profile in about 30 seconds. Not bad, eh?

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/56773c7f-2be4-4b10-a36a-53d29aacc585/3.png)

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/58751a37-082e-43c9-addb-cb47ff6959b1/4.png)

### Upcoming feature?

One piece of information I found interesting in chatting with Microsoft folks, was that this feature has been in private preview for some time now. Funnily enough, I recall about 4 days last summer when the “Migrate” button was present and we could do something similar. But then it was taken away. I imagine someone flipped a switch that should not have been switched.

If the Microsoft feature does make it’s way to Intune, then great! I’ll be the first one cheering. However, usually when I hear a feature is coming, I think of how far we currently are from Christmas, as that too is on the way.

At least in the meantime, we can start converting now.
