---
author: steve@getrubix.com
date: Thu, 26 Oct 2023 22:56:37 +0000
description: '"You’ve probably noticed by now that a lot of the action about the Intune
  device migration solution is happening on the GetRubix YouTube channel. It’s not
  because I don’t love rambling here, but a lot of these concepts are just easier
  to show, rather than tell."'
slug: update-about-updates
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/update-about-updates_thumbnail.jpg
title: Update about Updates
---

You’ve probably noticed by now that a lot of the action about the Intune device migration solution is happening on the [GetRubix YouTube](https://www.youtube.com/@getrubix9986) channel. It’s not because I don’t love rambling here, but a lot of these concepts are just easier to show, rather than tell.

Since we’ve been flinging updates out to this solution faster then you can say “bulk provisioning”, it seemed appropriate to put together a list of all the new features and components that have been added since the V1 solution. So here you go.

_\*Most of these are either already available in our_ [_GitHub repo_](https://github.com/stevecapacity/IntuneMigrationV2) _or will be added shortly._

Premigration
------------

In the V1, the entire migration process is run in one shot. This generally works for situations where user data is not an issue and we’re just moving between Azure (Entra) environments. However, there has been a demand to solve for the device existing in various source states, and the need to move larger amounts of user data. In order to plan the best workflows for different device scenarios, we needed a way collect data and analyze prior to performing the migration.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/f74d55ed-0692-469a-a9a6-e84ebc52d7d1/T2T+Migration+-+Page+9.png)

Now, we run a ‘premigration’ script that does just that. We’re using Azure blob storage to capture device information, determine the current device state, and calculate how much user data needs to be migrated. Then, if need be, we can use that blob storage to back up the user data so we’re not taking up additional storage on the device.

New Hardware
------------

When I first heard the ask to migrate between tenants _with_ a new PC, I thought, “why”? In theory, that should be the easiest path as you can just deploy a brand new Autopilot device and have the user sign in with destination tenant credentials.

But wait- ah yes, the user data. We would need a streamlined way to maintain files, folder, and most importantly, application data like preferences and settings. So using the blob storage method, we can pull down the information on a new device and restore the data to the new user profile. Unless…

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/52351530-0c88-4b07-bcce-d76a33a057aa/T2T+Migration+-+Page+9+%281%29.png)

Peer-to-Peer
------------

Two months ago, I was asked if there’s a way to migrate user data directly to a new PC in the destination tenant from the old PC in the source tenant over the user’s local network. This was intriguing.

We would have to take advantage of our premigration script and stage a file share with brand new credentials on the local machine that have read rights to that data. Once captured, we store the information in blob storage. When the new PC is deployed through Autopilot, it can retrieve the old device info, and then proceed to create a mapped network drive and move over the user data. And as long as the user left the original PC on and connected to the network, it worked!

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/3cf9668e-2a09-4767-a901-9b302c92d5dc/T2T+Migration+-+Hardware+Migration.png)

The Ghost Account
-----------------

Not all updates are sunshine, rainbows, and cotton candy. We have to fix the things that are broken. And the most broken part of the solution was after the migration. When all was moved over, if you peak in the account settings, you would most likely still see the source account just sitting there like this.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/960de3b4-f7e0-45b9-a5b3-013efe274530/Screenshot+2023-10-26+183738.png)

Even though it was harmless and not really connected to anything, we couldn’t just leave it there. That’s gross.

The solution ended up being fairly simple. You see, those are considered MSAL accounts, and they are stored in a very special file located at **%LOCALAPPDATA%\\Packages\\Microsoft.AAD.BrokerPlugin\_\*s0meR3alLyl0ngGU!dG03Shere**.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/9c579d02-38fa-43cb-a003-8d4461815563/Screenshot+2023-10-26+180940.png)

We tried deleting this file in between tenants, but no luck. The answer turned out to come from a suggestion by one of our Discord community members, @Karlmit. Since it lives in one of the _AppData_ locations we backup, all that had to be done was exclude it from the robocopy function:

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/11b2f890-81dd-47e8-a105-def12b02718f/Screenshot+2023-10-26+185320.png)

Domain Unjoin
-------------

While the V1 only accounted for Azure AD joined devices, we can now remove devices that are domain joined. This is handled via a check for **DomainJoined** status via _dsregcmd /status_ output, and then we just inject temporary credentials so the script can perform the unjoin.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/62a5caeb-ba93-42f7-befa-6c677cf132ba/Screenshot+2023-10-26+181535.png)

Toast
-----

I love toast. Not just the crunchy bread with butter or jelly, but the Windows toast notifications. Toast notifications have always been a struggle to run as system, because they really need to be executed in user context. But now, with the help of the [_RunAsUser_](https://github.com/KelvinTegelaar/RunAsUser) and [_BurntToast_](https://github.com/Windos/BurntToast) PowerShell modules, we can build in some custom toast notifications to key areas of the process where we want to communicate with our end users.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/133a5bbb-5641-4f87-b346-fad3b85b716b/Screenshot+2023-10-25+193242.png)

What’s next?
------------

This is just the start. The more we use this solution, the more needs we uncover, which then leads to more features. Here’s a look at our short term list that we’re working on next:

-   Improved migration logging
    
-   Migrating from MECM (SCCM) and co-managed device states
    
-   Alternate security options for source code, including secrets stored in key vaults and device certificates
    
-   Real time progress tracking visible in the toast notifications
    
-   Premigration data stored in an active Cosmos DB database for review
    

If you’ve been testing this solution, then thank you and let us know what feedback you have and ideas you’d like to see implemented in the future.

And if you want to chat, come see us over at the new [GetRubix Discord server](https://discord.gg/getrubix).
