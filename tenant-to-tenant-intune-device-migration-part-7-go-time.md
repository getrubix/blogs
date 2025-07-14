---
author: steve@getrubix.com
date: Mon, 07 Aug 2023 18:15:50 +0000
description: '"Today we’re going through the entire migration process, step-by-step.
  If you’ve been following along, you should have all the required files including
  the scripts, tasks, and provisioning package. We’re going to deploy the migration
  app with Intune in Tenant A, and then watch the migration to"'
slug: tenant-to-tenant-intune-device-migration-part-7-go-time
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/tenant-to-tenant-intune-device-migration-part-7-go-time_thumbnail.jpg
title: Tenant to Tenant Intune Device Migration Part 7 Go Time
---

Today we’re going through the entire migration process, step-by-step. If you’ve been following along, you should have all the required files including the scripts, tasks, and provisioning package. We’re going to deploy the migration app with Intune in Tenant A, and then watch the migration to Tenant B.

All the files are now available in the [Intune migration repository, here](https://github.com/stevecapacity/IntuneMigration).

Wrap it up
----------

Place all of your migration files in one directory, and then use the [Microsoft Win32 Content Prep Tool](https://github.com/Microsoft/Microsoft-Win32-Content-Prep-Tool) to create the .intunewin app package.

![Screenshot 2023-08-07 104621.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420107212-AT8NRM1Y9JZB5P6BEQ30/Screenshot+2023-08-07+104621.png)

![Screenshot 2023-08-07 104621.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420107212-AT8NRM1Y9JZB5P6BEQ30/Screenshot+2023-08-07+104621.png)

![Screenshot 2023-08-07 105444.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420107170-9YRMOM4E9A2YNMM7KIQJ/Screenshot+2023-08-07+105444.png)

![Screenshot 2023-08-07 105444.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420107170-9YRMOM4E9A2YNMM7KIQJ/Screenshot+2023-08-07+105444.png)

#block-yui\_3\_17\_2\_1\_1691419642561\_16830 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -20px; } #block-yui\_3\_17\_2\_1\_1691419642561\_16830 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 20px; margin-bottom: 20px; }

Upload the app to Intune in Tenant A and use the following for the install command:

```
powershell.exe -executionpolicy bypass .\StartMigrate.ps1
```

For the detection rule, just use the flag we create in the **StartMigrate** script.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/118516d4-4b54-4413-8c17-db2b33771dcd/Screenshot+2023-08-07+105903.png)

Set the assignment as available to a user group. In my case, I’m using a user group called “Migration Users” and just adding members manually.

_\*For step by step instructions on packaging an Intune Windows app,_ [_read here_](https://www.getrubix.com/blog/app-answers-yes-intune-can-do-it)_._

The existing device
-------------------

Let’s take a look at the state of our current device in Tenant A, logged in as **_steve.weiner@rubixdev.com_**.

[View fullsize

![Logon screen](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420693656-R1HKB8O8MVZZNJ96UJ1I/Screenshot+2023-08-04+213817.png)

![Logon screen](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420693656-R1HKB8O8MVZZNJ96UJ1I/Screenshot+2023-08-04+213817.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420693656-R1HKB8O8MVZZNJ96UJ1I/Screenshot+2023-08-04+213817.png)

Logon screen

[View fullsize

![Desktop](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420694832-XWQ24PKD8OOSDGXMY9P7/Screenshot+2023-08-04+215133.png)

![Desktop](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420694832-XWQ24PKD8OOSDGXMY9P7/Screenshot+2023-08-04+215133.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420694832-XWQ24PKD8OOSDGXMY9P7/Screenshot+2023-08-04+215133.png)

Desktop

[View fullsize

![App settings: VS Code](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420695039-F3DUDMAPW8ZU73BBT434/Screenshot+2023-08-04+215212.png)

![App settings: VS Code](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420695039-F3DUDMAPW8ZU73BBT434/Screenshot+2023-08-04+215212.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420695039-F3DUDMAPW8ZU73BBT434/Screenshot+2023-08-04+215212.png)

App settings: VS Code

[View fullsize

![Chrome](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420693598-PJHYTIPU8GTQF3OGIBI3/Screenshot+2023-08-04+215307.png)

![Chrome](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420693598-PJHYTIPU8GTQF3OGIBI3/Screenshot+2023-08-04+215307.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691420693598-PJHYTIPU8GTQF3OGIBI3/Screenshot+2023-08-04+215307.png)

Chrome

#block-yui\_3\_17\_2\_1\_1691419642561\_34765 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -15px; } #block-yui\_3\_17\_2\_1\_1691419642561\_34765 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 15px; margin-bottom: 15px; }

_Click on each image for more info._

Essentially, you can see the lock screen, signed in account, desktop shortcuts, and app settings- all things you would expect from a PC in use. To begin the migration, we’ll launch the Company Portal.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/16451d75-784b-4892-bd85-bf0bf6207f72/Migration+App.png)

An end user can click “Install” to start the process. At this point, the **StartMigrate** script will run, unpacking it’s contents and going through the pieces we’ve compiled. When it’s finished, the user will see a notification about being signed out, followed by an automatic reboot.

[View fullsize

![Migration App2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1741976566713-TXWTQ840ICUVZ83XSLCX/Migration+App2.png)

![Migration App2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1741976566713-TXWTQ840ICUVZ83XSLCX/Migration+App2.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1741976566713-TXWTQ840ICUVZ83XSLCX/Migration+App2.png)

[View fullsize

![Screenshot 2023-08-04 215602.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427434960-LXGOEMYZDVDPT87EGOSZ/Screenshot+2023-08-04+215602.png)

![Screenshot 2023-08-04 215602.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427434960-LXGOEMYZDVDPT87EGOSZ/Screenshot+2023-08-04+215602.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427434960-LXGOEMYZDVDPT87EGOSZ/Screenshot+2023-08-04+215602.png)

#block-yui\_3\_17\_2\_1\_1691419642561\_120795 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -10px; } #block-yui\_3\_17\_2\_1\_1691419642561\_120795 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 10px; margin-bottom: 10px; }

We’re not in Tenant A anymore…
------------------------------

When it starts back up, the device will sit for 30 seconds, followed by another automatic reboot. Notice the lock screen image has been reset to default.

[View fullsize

![Screenshot 2023-08-04 215625.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691428738073-IW9MCOFDA0OY13C51Z90/Screenshot+2023-08-04+215625.png)

![Screenshot 2023-08-04 215625.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691428738073-IW9MCOFDA0OY13C51Z90/Screenshot+2023-08-04+215625.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691428738073-IW9MCOFDA0OY13C51Z90/Screenshot+2023-08-04+215625.png)

[View fullsize

![Screenshot 2023-08-04 215648.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691428738091-IZNBKCKZ7X03TB7U5D3Q/Screenshot+2023-08-04+215648.png)

![Screenshot 2023-08-04 215648.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691428738091-IZNBKCKZ7X03TB7U5D3Q/Screenshot+2023-08-04+215648.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691428738091-IZNBKCKZ7X03TB7U5D3Q/Screenshot+2023-08-04+215648.png)

#block-yui\_3\_17\_2\_1\_1691419642561\_169108 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -10px; } #block-yui\_3\_17\_2\_1\_1691419642561\_169108 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 10px; margin-bottom: 10px; }

Once the device comes back after the 2nd reboot, you can see all of the Tenant B policy coming over from Intune. We’re still keep the “Other user” setting so there’s no mistake that a user has to log in with Tenant B credentials. I’m now signing in as **_steve.weiner@stevecapacity.com_**.

[View fullsize

![Screenshot 2023-08-04 215700.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427611649-A2ZRLK55T5J8ZVK9DNPI/Screenshot+2023-08-04+215700.png)

![Screenshot 2023-08-04 215700.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427611649-A2ZRLK55T5J8ZVK9DNPI/Screenshot+2023-08-04+215700.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427611649-A2ZRLK55T5J8ZVK9DNPI/Screenshot+2023-08-04+215700.png)

[View fullsize

![Screenshot 2023-08-04 215710.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427613086-2KIZ34KAYHU11P9D4KUV/Screenshot+2023-08-04+215710.png)

![Screenshot 2023-08-04 215710.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427613086-2KIZ34KAYHU11P9D4KUV/Screenshot+2023-08-04+215710.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427613086-2KIZ34KAYHU11P9D4KUV/Screenshot+2023-08-04+215710.png)

[View fullsize

![Screenshot 2023-08-04 215753.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427611571-A4NFX0CVEE1OVPC2SM0L/Screenshot+2023-08-04+215753.png)

![Screenshot 2023-08-04 215753.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427611571-A4NFX0CVEE1OVPC2SM0L/Screenshot+2023-08-04+215753.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691427611571-A4NFX0CVEE1OVPC2SM0L/Screenshot+2023-08-04+215753.png)

#block-yui\_3\_17\_2\_1\_1691419642561\_139196 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -8px; } #block-yui\_3\_17\_2\_1\_1691419642561\_139196 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 8px; margin-bottom: 8px; }

At the desktop, more things are already kicking in. We can see the new user credentials in the account settings along with the updated desktop background image. Even though all my user data has not come over yet, I can get right to work with Microsoft 365 apps like Outlook.

Before the next step, we’ll check Intune in Tenant B. It shows us the device is enrolled in, but the primary user is empty- this is because our script didn’t run yet.

[View fullsize

![Screenshot 2023-08-04 215838.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691429401406-RW0BC2AUGXFWAB1VG06Z/Screenshot+2023-08-04+215838.png)

![Screenshot 2023-08-04 215838.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691429401406-RW0BC2AUGXFWAB1VG06Z/Screenshot+2023-08-04+215838.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691429401406-RW0BC2AUGXFWAB1VG06Z/Screenshot+2023-08-04+215838.png)

[View fullsize

![Screenshot 2023-08-04 215901.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691429402549-T4T6KT8YJLD7Q7R18S89/Screenshot+2023-08-04+215901.png)

![Screenshot 2023-08-04 215901.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691429402549-T4T6KT8YJLD7Q7R18S89/Screenshot+2023-08-04+215901.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691429402549-T4T6KT8YJLD7Q7R18S89/Screenshot+2023-08-04+215901.png)

[View fullsize

![Screenshot 2023-08-04 220259.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691429475538-8GILE8O9KV0QFDJ1UKHJ/Screenshot+2023-08-04+220259.png)

![Screenshot 2023-08-04 220259.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691429475538-8GILE8O9KV0QFDJ1UKHJ/Screenshot+2023-08-04+220259.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691429475538-8GILE8O9KV0QFDJ1UKHJ/Screenshot+2023-08-04+220259.png)

#block-yui\_3\_17\_2\_1\_1691419642561\_179664 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -10px; } #block-yui\_3\_17\_2\_1\_1691419642561\_179664 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 10px; margin-bottom: 10px; }

Bringing it all back…
---------------------

After the **RestoreProfile** script runs, we will see the prompt for the last reboot. But before that happens, check out the desktop folder which has been restored from the original profile.

_\*I scrapped the wallpaper so it would be easier to see everything, as the awesome 80’s vibe makes things a little busy._

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/0ae66f9b-db0d-47e2-b5cf-5ab4ad0d937b/Screenshot+2023-08-04+220323.png)

After the last reboot, you can see we’ve re-enabled allowing the last signed-in user to be displayed at the lock screen.

![](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/e0cb1f91-b88c-4203-b947-8cc1bfe28534/Screenshot+2023-08-04+220359.png)

From here on out, things are pretty smooth. All the files have returned, including the _AppData_ folders. When I launch VS Code and Chrome, all settings have come back. You can even see the account still present in Chrome, the sync will just be paused until I sign in again.

[View fullsize

![Screenshot 2023-08-04 220640.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430170431-83ERUWBGBAT83NTTRMXD/Screenshot+2023-08-04+220640.png)

![Screenshot 2023-08-04 220640.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430170431-83ERUWBGBAT83NTTRMXD/Screenshot+2023-08-04+220640.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430170431-83ERUWBGBAT83NTTRMXD/Screenshot+2023-08-04+220640.png)

[View fullsize

![Screenshot 2023-08-04 220650.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430170359-ZX2DVMKTV527WWTG01KM/Screenshot+2023-08-04+220650.png)

![Screenshot 2023-08-04 220650.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430170359-ZX2DVMKTV527WWTG01KM/Screenshot+2023-08-04+220650.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430170359-ZX2DVMKTV527WWTG01KM/Screenshot+2023-08-04+220650.png)

#block-yui\_3\_17\_2\_1\_1691419642561\_195552 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -10px; } #block-yui\_3\_17\_2\_1\_1691419642561\_195552 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 10px; margin-bottom: 10px; }

After a bit, we can go back to Intune and see the primary user has been updated, the BitLocker recovery key is there, and we’re registered in Tenant B Autopilot _with_ the proper group tag.

You can even log into the Azure portal and see the device associated with the appropriate user and Intune device objects.

[View fullsize

![Screenshot 2023-08-04 221418.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430289473-GKBRAMT8NHC4VF5WBQ1U/Screenshot+2023-08-04+221418.png)

![Screenshot 2023-08-04 221418.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430289473-GKBRAMT8NHC4VF5WBQ1U/Screenshot+2023-08-04+221418.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430289473-GKBRAMT8NHC4VF5WBQ1U/Screenshot+2023-08-04+221418.png)

[View fullsize

![Screenshot 2023-08-04 221524.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430296654-OT8UVS0LZFY5NA1NLO41/Screenshot+2023-08-04+221524.png)

![Screenshot 2023-08-04 221524.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430296654-OT8UVS0LZFY5NA1NLO41/Screenshot+2023-08-04+221524.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430296654-OT8UVS0LZFY5NA1NLO41/Screenshot+2023-08-04+221524.png)

[View fullsize

![Screenshot 2023-08-04 221554.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430310633-6XFQIDQC745D5IR2NEB7/Screenshot+2023-08-04+221554.png)

![Screenshot 2023-08-04 221554.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430310633-6XFQIDQC745D5IR2NEB7/Screenshot+2023-08-04+221554.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430310633-6XFQIDQC745D5IR2NEB7/Screenshot+2023-08-04+221554.png)

[View fullsize

![Screenshot 2023-08-04 222032.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430313079-2GN5O9Q0TRTEJHBSXDN2/Screenshot+2023-08-04+222032.png)

![Screenshot 2023-08-04 222032.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430313079-2GN5O9Q0TRTEJHBSXDN2/Screenshot+2023-08-04+222032.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430313079-2GN5O9Q0TRTEJHBSXDN2/Screenshot+2023-08-04+222032.png)

#block-yui\_3\_17\_2\_1\_1691419642561\_207738 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -3px; } #block-yui\_3\_17\_2\_1\_1691419642561\_207738 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 3px; margin-bottom: 3px; }

Check the logs…
---------------

This process was fairly painless, but things can go wrong. If a script didn’t run, or something didn’t behave the way it should, navigate to **C:\\ProgramData\\IntuneMigration** and you should see a _migration.log_ and _post-migration.log_ file.

The _migraiton.log_ covers everything leading up to the Tenant A exit. _Post-migration_ is created the moment after the first reboot. In here, you can see everything happening including the tasks running, data migration, Azure leave, etc.

[View fullsize

![Screenshot 2023-08-04 220933.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430692374-O50E6L0YGE9621CIYVGN/Screenshot+2023-08-04+220933.png)

![Screenshot 2023-08-04 220933.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430692374-O50E6L0YGE9621CIYVGN/Screenshot+2023-08-04+220933.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430692374-O50E6L0YGE9621CIYVGN/Screenshot+2023-08-04+220933.png)

[View fullsize

![Screenshot 2023-08-04 221304.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430692391-KES915OMZ6Q4X1XB34J5/Screenshot+2023-08-04+221304.png)

![Screenshot 2023-08-04 221304.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430692391-KES915OMZ6Q4X1XB34J5/Screenshot+2023-08-04+221304.png)](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1691430692391-KES915OMZ6Q4X1XB34J5/Screenshot+2023-08-04+221304.png)

#block-yui\_3\_17\_2\_1\_1691419642561\_234126 .sqs-gallery-block-grid .sqs-gallery-design-grid { margin-right: -10px; } #block-yui\_3\_17\_2\_1\_1691419642561\_234126 .sqs-gallery-block-grid .sqs-gallery-design-grid-slide .margin-wrapper { margin-right: 10px; margin-bottom: 10px; }

Are we done?
------------

So is that it? Well, yes and no. As of now this process works great for migrating a PC from Azure AD to Azure AD. But there are some more things in the works, including:

-   Hybrid Azure AD to Azure AD
    
-   Domain Joined to Azure AD
    
-   Co-Managed to Intune
    
-   SCCM only to Intune
    
-   Tenant to Tenant migration _WITH_ new hardware
    

And that’s just to name a few. I’m also working on a video walkthrough that should be on our [YouTube channel](https://www.youtube.com/@getrubix9986) in the next week or so.

Also, you’ll notice in most of the screen grabs from my Virtual Machine, I left the time visible. We started at about 6:51pm and ended by 7:12pm. From an end-user perspective, that’ just about 20 minutes; not too shabby.

Thanks for following me through this and have fun migrating!
