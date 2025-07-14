---
author: steve@getrubix.com
date: Mon, 27 Jun 2022 12:29:36 +0000
description: '"Windows 11 is without a doubt, my favorite version of the OS so far.
  But, it’s still Windows, which means there are a few annoying things you’re going
  to encounter along the way. During my first experience with “11”, I was surprised
  about that built-in consumer"'
slug: be-gone-chat
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/be-gone-chat_thumbnail.jpg
title: Be gone chat
---

Windows 11 is without a doubt, my favorite version of the OS so far. But, it’s still Windows, which means there are a few annoying things you’re going to encounter along the way. During my first experience with “11”, I was surprised about that built-in consumer version of Teams, chat, home-Teams… whatever it’s supposed to be called (turns out it’s both “Microsoft Teams” _and_ “Chat”; not confusing at all).

Not interested in the least, I deleted it. Fast forward to major enterprises starting to deploy Windows 11, and it turns out, simply deleting is not that simple.

The answer was somewhat complicated in that it required a registry edit that Intune could not natively execute, due to a permissions issue. Rather than drone on about it here, we made a quick screencast walking you through the process.

If you want the PowerShell script that was used, you can grab it [here](https://github.com/groovemaster17/IntunePowershell/blob/main/removeChat.ps1). Enjoy!
